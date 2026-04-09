# pytest

## Standard output
```
============================= test session starts ==============================
platform linux -- Python 3.12.2, pytest-8.1.1, pluggy-1.4.0
rootdir: /home/user/project
configfile: pyproject.toml
plugins: cov-5.0.0, asyncio-0.23.6, mock-3.14.0
collected 37 items

tests/test_auth.py ....F.                                               [ 16%]
tests/test_models.py ........                                            [ 37%]
tests/test_api.py ...F..F...                                             [ 64%]
tests/test_db.py .............                                           [ 100%]

=================================== FAILURES ===================================
_________________________________ test_auth.py::test_token_expiry ______________

    def test_token_expiry():
        token = create_token(user_id=1, expires_in=-1)
>       result = verify_token(token)
E       AssertionError: assert result.is_valid == False
E        +  where result.is_valid = TokenResult(is_valid=True, user_id=1, error=None).is_valid

tests/test_auth.py:45: AssertionError
_________________________________ test_api.py::test_rate_limit _________________

    @pytest.mark.asyncio
    async def test_rate_limit():
        client = AsyncClient(app=app, base_url="http://test")
        responses = []
        for i in range(110):
            resp = await client.get("/api/data")
            responses.append(resp)
>       assert responses[-1].status_code == 429
E       AssertionError: assert 200 == 429
E        +  where 200 = <Response [200 OK]>.status_code

tests/test_api.py:67: AssertionError
_________________________________ test_api.py::test_pagination_overflow ________

    def test_pagination_overflow():
        response = client.get("/api/items?page=99999")
>       assert response.status_code == 200
E       assert 500 == 200
E        +  where 500 = <Response [500 Internal Server Error]>.status_code

During handling of the above exception, another exception occurred:

    def test_pagination_overflow():
        response = client.get("/api/items?page=99999")
        assert response.status_code == 200
>       data = response.json()
E       json.decoder.JSONDecodeError: Expecting value: line 1 column 1 (char 0)

tests/test_api.py:90: JSONDecodeError
=========================== short test summary info ============================
FAILED tests/test_auth.py::test_token_expiry - AssertionError: assert result....
FAILED tests/test_api.py::test_rate_limit - AssertionError: assert 200 == 429
FAILED tests/test_api.py::test_pagination_overflow - json.decoder.JSONDecodeError
========================= 3 failed, 34 passed in 2.41s ========================
```

## `--llm` output
```
FAIL 3/37

--- tests/test_auth.py:45 test_token_expiry
AssertionError: assert result.is_valid == False
  result.is_valid = True (TokenResult(is_valid=True, user_id=1, error=None))

--- tests/test_api.py:67 test_rate_limit
AssertionError: assert 200 == 429
  expected status_code 429, got 200

--- tests/test_api.py:89 test_pagination_overflow
assert 500 == 200 (GET /api/items?page=99999)
  then json.decoder.JSONDecodeError: Expecting value: line 1 column 1 (char 0)
  at tests/test_api.py:90
```

## Design decisions
| Decision | Rationale |
|---|---|
| No session header (platform, plugins, rootdir) | Environment metadata is irrelevant to diagnosing test failures |
| No progress dots or percentage bar | Agent doesn't need a visual progress indicator |
| No `=` separator lines | Pure decoration; wastes tokens |
| No source code context in tracebacks | Agent can read the file directly; inline code snippets duplicate information |
| pytest's `where` introspection preserved as compact detail | Shows the actual vs expected values that the agent needs to craft a fix |
| Chained exceptions collapsed into `then` | The pagination test has two errors — both matter, but the causal chain is expressed in one line instead of a full second traceback block |
| No short test summary section | Redundant — each failure is already listed above with full detail |
