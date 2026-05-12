---
layout: page
title: Temporal.io with Python
parent: Task Automation
description: Using Temporal.io to build durable, fault-tolerant workflows in Python
nav_order: 4
tags: [python, temporal, workflows, task-automation, orchestration]
---

# Temporal.io with Python: Durable Workflows That Actually Survive Failure
{:.no_toc}

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

Most task queues (Celery, RQ, etc.) will happily lose your job if the worker crashes mid-execution. Temporal takes a different approach: it persists the entire execution state of your workflow, so if a worker dies halfway through a 10-step pipeline, it resumes from exactly where it left off — not from the beginning.

---

## Core Concepts

Before writing any code, four terms are worth pinning down:

- **Workflow**: A durable function that orchestrates the overall logic. It must be deterministic — no random numbers, no `datetime.now()`, no direct I/O. Temporal replays it from event history.
- **Activity**: A regular function that does the actual work (API calls, DB writes, file processing). Activities *can* have side effects and are what Temporal retries on failure.
- **Worker**: A process that polls Temporal's server for tasks and executes workflows and activities.
- **Task Queue**: A named channel that connects workflows/activities to workers.

The key insight: **workflows are pure orchestration, activities are the side-effecting units**. This separation is what makes replay and retry safe.

---

## Installation and Setup

Install the Python SDK:

```bash
pip install temporalio
```

You also need a running Temporal server. The easiest way for local development:

```bash
# Using Docker
docker run --rm -it \
  -p 7233:7233 \
  -p 8080:8080 \
  temporalio/auto-setup:latest
```

The Temporal Web UI will be available at `http://localhost:8080`.

---

## Your First Workflow

Let's build a simple order processing pipeline: validate an order, charge a customer, and send a confirmation email.

### Define the Activities

Activities are plain async functions decorated with `@activity.defn`. Each one does exactly one thing and is independently retryable.

```python
# activities.py
import asyncio
from temporalio import activity

@activity.defn
async def validate_order(order_id: str) -> bool:
    activity.logger.info(f"Validating order {order_id}")
    await asyncio.sleep(0.5)  # Simulates DB lookup
    return True

@activity.defn
async def charge_customer(order_id: str, amount: float) -> str:
    activity.logger.info(f"Charging {amount} for order {order_id}")
    await asyncio.sleep(1.0)  # Simulates payment API call
    return f"charge_{order_id}_ok"

@activity.defn
async def send_confirmation(order_id: str, charge_id: str) -> None:
    activity.logger.info(f"Sending confirmation for {order_id} (charge: {charge_id})")
    await asyncio.sleep(0.3)  # Simulates email API
```

### Define the Workflow

The workflow calls activities via `workflow.execute_activity()`. Notice: **no direct I/O here** — only calls to activities.

```python
# workflows.py
from datetime import timedelta
from temporalio import workflow
from temporalio.common import RetryPolicy

# Import activities via sandbox-safe wrapper
with workflow.unsafe.imports_passed_through():
    from activities import validate_order, charge_customer, send_confirmation

@workflow.defn
class OrderWorkflow:
    @workflow.run
    async def run(self, order_id: str, amount: float) -> str:
        retry_policy = RetryPolicy(maximum_attempts=3)

        # Step 1: Validate
        is_valid = await workflow.execute_activity(
            validate_order,
            order_id,
            start_to_close_timeout=timedelta(seconds=10),
            retry_policy=retry_policy,
        )

        if not is_valid:
            raise ValueError(f"Order {order_id} failed validation")

        # Step 2: Charge
        charge_id = await workflow.execute_activity(
            charge_customer,
            args=[order_id, amount],
            start_to_close_timeout=timedelta(seconds=30),
            retry_policy=retry_policy,
        )

        # Step 3: Confirm
        await workflow.execute_activity(
            send_confirmation,
            args=[order_id, charge_id],
            start_to_close_timeout=timedelta(seconds=10),
            retry_policy=retry_policy,
        )

        return charge_id
```

### Run the Worker

The worker registers which workflows and activities it can handle, then polls Temporal for work.

```python
# worker.py
import asyncio
from temporalio.client import Client
from temporalio.worker import Worker
from workflows import OrderWorkflow
from activities import validate_order, charge_customer, send_confirmation

async def main():
    client = await Client.connect("localhost:7233")

    worker = Worker(
        client,
        task_queue="order-processing",
        workflows=[OrderWorkflow],
        activities=[validate_order, charge_customer, send_confirmation],
    )

    print("Worker started, polling for tasks...")
    await worker.run()

if __name__ == "__main__":
    asyncio.run(main())
```

### Trigger a Workflow Execution

```python
# run_workflow.py
import asyncio
from temporalio.client import Client
from workflows import OrderWorkflow

async def main():
    client = await Client.connect("localhost:7233")

    result = await client.execute_workflow(
        OrderWorkflow.run,
        args=["order-42", 99.99],
        id="order-42-workflow",
        task_queue="order-processing",
    )

    print(f"Workflow completed. Charge ID: {result}")

if __name__ == "__main__":
    asyncio.run(main())
```

Run the worker in one terminal, then `run_workflow.py` in another. You'll see each activity log its progress in the worker output.

---

## Retry Policies

One of Temporal's most practical features is granular retry control per activity. You can set different policies for each step:

```python
from temporalio.common import RetryPolicy
from datetime import timedelta

# Aggressive retry for a flaky external API
flaky_api_retry = RetryPolicy(
    initial_interval=timedelta(seconds=1),
    backoff_coefficient=2.0,       # Exponential backoff
    maximum_interval=timedelta(minutes=2),
    maximum_attempts=10,
    non_retryable_error_types=["ValueError"],  # Don't retry on bad input
)

# Conservative retry for an expensive payment call
payment_retry = RetryPolicy(
    initial_interval=timedelta(seconds=5),
    maximum_attempts=3,
    non_retryable_error_types=["PaymentDeclinedError"],
)

charge_id = await workflow.execute_activity(
    charge_customer,
    args=[order_id, amount],
    start_to_close_timeout=timedelta(seconds=30),
    retry_policy=payment_retry,
)
```

If an activity exhausts all retries, Temporal marks the workflow as failed with the last exception — and you can inspect the full history in the Web UI.

---

## Timers and Long-Running Workflows

Temporal handles long-running workflows natively. A workflow can sleep for days without holding any resources, because `workflow.sleep()` is backed by Temporal's event history, not a real OS sleep.

```python
@workflow.defn
class SubscriptionWorkflow:
    @workflow.run
    async def run(self, user_id: str) -> None:
        # Send a welcome email immediately
        await workflow.execute_activity(
            send_welcome_email,
            user_id,
            start_to_close_timeout=timedelta(seconds=10),
        )

        # Wait 7 days — worker can restart, server can restart, doesn't matter
        await workflow.sleep(timedelta(days=7))

        # Send a follow-up after the trial period
        await workflow.execute_activity(
            send_trial_ending_email,
            user_id,
            start_to_close_timeout=timedelta(seconds=10),
        )

        await workflow.sleep(timedelta(days=1))

        await workflow.execute_activity(
            convert_to_paid_or_cancel,
            user_id,
            start_to_close_timeout=timedelta(seconds=30),
        )
```

This workflow survives restarts, deployments, and crashes during any of those 8 days.

---

## Signals and Queries

Workflows aren't black boxes — you can send them signals (to change behaviour mid-run) or query them (to read their current state).

```python
@workflow.defn
class ApprovalWorkflow:
    def __init__(self) -> None:
        self._approved: bool = False
        self._rejection_reason: str = ""

    @workflow.signal
    async def approve(self) -> None:
        self._approved = True

    @workflow.signal
    async def reject(self, reason: str) -> None:
        self._rejection_reason = reason

    @workflow.query
    def status(self) -> str:
        if self._approved:
            return "approved"
        if self._rejection_reason:
            return f"rejected: {self._rejection_reason}"
        return "pending"

    @workflow.run
    async def run(self, request_id: str) -> str:
        # Wait until approved or rejected (up to 48 hours)
        await workflow.wait_condition(
            lambda: self._approved or bool(self._rejection_reason),
            timeout=timedelta(hours=48),
        )

        if self._approved:
            await workflow.execute_activity(
                process_approval,
                request_id,
                start_to_close_timeout=timedelta(seconds=30),
            )
            return "processed"

        return f"rejected: {self._rejection_reason}"
```

From the client side:

```python
# Send a signal to a running workflow
handle = client.get_workflow_handle("approval-workflow-123")
await handle.signal(ApprovalWorkflow.approve)

# Query its current state
status = await handle.query(ApprovalWorkflow.status)
print(status)  # "approved"
```

---

## Child Workflows

For complex orchestration, you can fan out into child workflows. Each child runs independently with its own history and retry behaviour.

```python
@workflow.defn
class BatchProcessingWorkflow:
    @workflow.run
    async def run(self, item_ids: list[str]) -> list[str]:
        # Launch all child workflows in parallel
        handles = [
            await workflow.start_child_workflow(
                ProcessItemWorkflow.run,
                item_id,
                id=f"process-item-{item_id}",
                task_queue="item-processing",
            )
            for item_id in item_ids
        ]

        # Wait for all to complete
        results = await asyncio.gather(*[handle for handle in handles])
        return results
```

Each child has independent failure/retry semantics — if one fails, others keep running.

---

## Testing Workflows

The SDK includes a `WorkflowEnvironment` for testing without a real Temporal server. Time can be skipped programmatically, making tests for long-running workflows fast.

```python
import pytest
from temporalio.testing import WorkflowEnvironment
from temporalio.worker import Worker
from workflows import OrderWorkflow
from activities import validate_order, charge_customer, send_confirmation

@pytest.mark.asyncio
async def test_order_workflow_success():
    async with await WorkflowEnvironment.start_time_skipping() as env:
        async with Worker(
            env.client,
            task_queue="test-orders",
            workflows=[OrderWorkflow],
            activities=[validate_order, charge_customer, send_confirmation],
        ):
            result = await env.client.execute_workflow(
                OrderWorkflow.run,
                args=["test-order-1", 49.99],
                id="test-order-1",
                task_queue="test-orders",
            )

        assert result.startswith("charge_test-order-1")
```

For activity-level testing, mock them directly:

```python
from unittest.mock import AsyncMock
from temporalio.testing import WorkflowEnvironment

@pytest.mark.asyncio
async def test_order_workflow_with_mocked_activities():
    mock_validate = AsyncMock(return_value=True)
    mock_charge = AsyncMock(return_value="charge_mock_ok")
    mock_confirm = AsyncMock(return_value=None)

    async with await WorkflowEnvironment.start_time_skipping() as env:
        async with Worker(
            env.client,
            task_queue="test-orders",
            workflows=[OrderWorkflow],
            activities=[mock_validate, mock_charge, mock_confirm],
        ):
            result = await env.client.execute_workflow(
                OrderWorkflow.run,
                args=["order-99", 19.99],
                id="order-99",
                task_queue="test-orders",
            )

        assert result == "charge_mock_ok"
        mock_validate.assert_called_once_with("order-99")
        mock_charge.assert_called_once_with("order-99", 19.99)
```

---

## When to Reach for Temporal

Temporal adds operational complexity (you run a server), so it's worth being deliberate about when to use it:

| Situation | Good fit? |
|---|---|
| Multi-step pipeline where partial failure is costly | ✅ Yes |
| Workflows that span hours, days, or weeks | ✅ Yes |
| Human-in-the-loop approval flows | ✅ Yes |
| Needs audit trail / full execution history | ✅ Yes |
| Simple background job that can safely retry from scratch | ⚠️ Maybe (Celery is simpler) |
| Fire-and-forget tasks with no state | ❌ Overkill |

The sweet spot is any workflow where "just retry the whole thing" is unacceptable — either because the earlier steps had side effects (payment already charged), because the workflow runs for too long, or because you need human input partway through.
