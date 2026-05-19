---
description: >-
  Learn which market parameters affect order placement on Yellow, including
  price increments, size increments, and minimum order requirements.
---

# Market Parameters

Each market on Yellow has a set of rules that define how orders can be placed. These parameters are shown in the market details and are used by the order form and platform checks.

### Trading Parameters

#### Tick Size (Price Step)

The tick size is the smallest allowed price increment for an order.

Example:

* if the tick size is `0.10`, valid prices include `100.0`, `100.1`, and `100.2`
* `100.05` would be invalid

#### Step Size (Size Step)

The step size is the smallest allowed increment for order size.

Example:

* if the step size is `0.001`, valid sizes include `0.001`, `0.002`, and `0.015`
* `0.0005` would be invalid

#### Minimum Order Size

This is the smallest order size allowed for a market.

Orders below this size will be rejected.

#### Minimum Order Value

Some markets may also require a minimum order value.

Order value is usually calculated as:

`price × size`

This means an order can meet the minimum size requirement but still be rejected if its total value is too low.

#### Maximum Order Size

Some markets may also enforce a maximum size for a single order.

### Why Orders Get Rejected

An order may be rejected if:

* the price is not aligned with the tick size
* the size is not aligned with the step size
* the size is below the minimum order size
* the order value is below the minimum order value
* the order exceeds the maximum order size, if enforced

### What to Check Before Placing an Order

Before submitting an order, check:

* the market symbol
* the tick size
* the step size
* the minimum order size
* the minimum order value, if used
* the maximum order size, if enforced
