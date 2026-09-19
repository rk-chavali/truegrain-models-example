# An example truegrain model

A small retail model in Apache Ossie, kept in a repository of its own so a
truegrain engine can follow it the way a team would actually run one: the
model is reviewed as a pull request, and a merge becomes the served model.

```
models/retail.yaml    orders, order_lines and customers
```

## Point an engine at it

```bash
truegrain serve rest -models git+https://github.com/rk-chavali/truegrain-models-example.git#main:models
```

The engine clones it, serves what it finds, and re-reads it on a timer.
`GET /v1/health` reports the commit being served, so a pipeline can merge and
then wait for `origin.commit` to move rather than guessing.

A pipeline holding the `deploy:model` scope can skip the wait:

```bash
curl -X POST https://your-engine/v1/reload -H "Authorization: Bearer $DEPLOY_TOKEN"
```

That means "look now", not "install this". There is no way to push a model
into an engine, which is the point: the repository is the only way in, so the
review, the checks and the diff cannot be stepped around.

## What the model is shaped to show

`orders` has a one-to-many child, `order_lines`, and a many-to-one parent,
`customers`:

```
order_lines  --many-to-one-->  orders  --many-to-one-->  customers
```

Grouping `order_revenue` by `customers.region` is a safe join and answers
normally. Grouping it by `order_lines.item_id` is not: joining `order_lines`
repeats every `orders` row once per line, so the total inflates from
**885.50** to **2361.00**.

truegrain refuses that question rather than answering it, and names the
metrics defined at the `order_lines` grain that do answer it correctly:
`line_revenue` and `units_sold`.

A model with no declared relationships cannot be told any of this, which is
why the relationships in `retail.yaml` are not decoration.

## Licence

Apache 2.0, the same as truegrain. Copy it, change it, point it at your own
tables.
