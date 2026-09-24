# Integration Evals Lab

A Python API client and a small tool-routing evaluation harness. The client uses [JSONPlaceholder](https://jsonplaceholder.typicode.com) for its public endpoints; the test suite uses an injected transport, so it runs without network access.

## API client

`IntegrationClient` validates upstream JSON into immutable `Post` and `User` models. It supports paginated reads, bearer-token headers, request timeouts, and bounded retries for GET requests. A POST requires an idempotency key before it can be retried. The key is sent in an `Idempotency-Key` header; JSONPlaceholder does not enforce it, so this example does not guarantee server-side deduplication.

The transport is injectable to keep retry and error-handling tests deterministic. HTTP 429 and 5xx responses can be retried; other 4xx responses fail immediately. Invalid JSON and unexpected payload shapes have separate errors.

## Routing evaluation

`integration-lab eval` checks whether short descriptions give a simple deterministic router enough information to select `list_posts`, `get_user`, or `create_post`. It includes nine fixed requests: reads, pagination, an explicit write, unsupported input, and ambiguous intent.

The sparse descriptions get 3/9 cases right; descriptions with explicit intent cues get 9/9. These are exact-match results on a small fixed suite, not a measure of general routing accuracy. The router matches phrases extracted from the descriptions.

## Run it

Python 3.10 or newer:

```bash
python -m venv .venv
. .venv/bin/activate
python -m pip install -e '.[dev]'
python -m pytest
```

To try the public API or run the evaluation:

```bash
integration-lab posts --page 1 --per-page 3
integration-lab user 1
integration-lab route "Show posts page 2"
integration-lab eval
```

`API_BEARER_TOKEN` is optional. JSONPlaceholder does not need one.

## Layout

```text
src/integration_lab/client.py      HTTP boundary, retries, validation
src/integration_lab/models.py      normalized domain objects
src/integration_lab/evaluation.py  routing cases and scoring
src/integration_lab/cli.py         command-line entry point
tests/                             transport and evaluation tests
```

There is no persistent store, OAuth flow, or server-side idempotency service. Writes against JSONPlaceholder are simulated by that service.
