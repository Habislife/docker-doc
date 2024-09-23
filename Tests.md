We want to run tests in an environment as similar as possible to production, so it only makes sense to do so inside of our containers!

## Test compose file

We will take advantage of the ability to specify multiple docker compose yaml files (as described in the previous lesson) to create a custom test configuration.

All we need to do is include commands to override the defaults that execute our test suite(s):

```yaml
services:
  api-node:
    command:
      - "npm"
      - "run"
      - "test"
  api-golang:
    command:
      - "go"
      - "test"
      - "-v"
      - "./..."
```

## Executing tests

To execute the tests we can then run the following docker compose commands:

```bash
docker-compose -f docker-compose-dev.yml -f docker-compose-test.yml run --rm api-node
docker-compose -f docker-compose-dev.yml -f docker-compose-test.yml run --rm api-golang
```

These commands will overlay the test configuration on the development configuration, run the tests for each service, and remove the containers after the tests are executed.