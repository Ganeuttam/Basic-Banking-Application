# Basic Banking API

A secure RESTful Banking API built with Python and FastAPI, featuring JWT authentication, account management, deposits, withdrawals, fund transfers, transaction history, and logout token blacklisting.

## 1. Project overview
A REST API for a basic banking application. Users can register, log in, view their account, deposit, withdraw, transfer money to another account, see transaction history, and log out. Protected endpoints require a JWT.

## 2. Technology stack
- Python 3.11+
- FastAPI
- SQLAlchemy 2.x with SQLite
- Pydantic v2
- PyJWT
- bcrypt
- pytest + httpx

## 3. Prerequisites
- Python 3.11 or newer
- Git
- VS Code (recommended)

## 4. Installation instructions

Windows PowerShell:
```powershell
python -m venv venv
venv\Scripts\activate
pip install -r requirements.txt
```

macOS/Linux:
```bash
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
```

## 5. Environment variables
Copy `.env.example` to `.env`.

Windows:
```powershell
copy .env.example .env
```

macOS/Linux:
```bash
cp .env.example .env
```

Variables:
| Variable | Meaning |
|---|---|
| DATABASE_URL | Database connection string |
| SECRET_KEY | Secret used to sign JWTs |
| ALGORITHM | JWT signing algorithm |
| ACCESS_TOKEN_EXPIRE_MINUTES | Token lifetime |

For a stronger local secret:
```bash
python -c "import secrets; print(secrets.token_hex(32))"
```

## 6. Database setup
No manual database setup is required. SQLite creates `bank.db` automatically when the application starts.

## 7. How to run the application
```bash
uvicorn app.main:app --reload
```

API: http://127.0.0.1:8000

## 8. API documentation
Swagger UI: http://127.0.0.1:8000/docs

ReDoc: http://127.0.0.1:8000/redoc

### Endpoints
| Method | Path | Auth | Description |
|---|---|---|---|
| POST | /api/auth/register | No | Register a user and create an account |
| POST | /api/auth/login | No | Get a JWT access token |
| POST | /api/auth/logout | Yes | Revoke the current token |
| GET | /api/accounts/me | Yes | Current user's account |
| POST | /api/accounts/deposit | Yes | Deposit money |
| POST | /api/accounts/withdraw | Yes | Withdraw money |
| POST | /api/accounts/transfer | Yes | Transfer to another account |
| GET | /api/accounts/transactions | Yes | Current user's transactions |

In Swagger, use `POST /api/auth/login`, copy `accessToken`, click **Authorize**, and paste only the token.

## 9. Authentication approach (including logout strategy)
Passwords are hashed with bcrypt and are never stored as plain text.

Login returns a signed JWT containing the user id (`sub`), token id (`jti`), issue time (`iat`) and expiry (`exp`).

Logout uses a token blacklist. The JWT's `jti` is stored in `blacklisted_tokens`, and every protected request checks the blacklist. Expired blacklist records are cleaned up.

## 10. Database design
- `users`: id, name, email, password_hash, created_at
- `accounts`: id, user_id, account_number, balance, created_at
- `transactions`: id, account_id, type, amount, status, reference_id, created_at
- `blacklisted_tokens`: id, jti, expires_at

One user has one account. Money is stored with `Numeric(12, 2)` rather than floating point. A database check constraint prevents negative balances.

## 11. Design decisions
- API routes handle HTTP and delegate business logic to services.
- Services contain banking rules.
- Repositories contain database queries.
- Transfers update both accounts and both history rows in one database transaction.
- Errors are converted into appropriate HTTP responses centrally.
- Login failures use the same message for unknown email and wrong password.
- Protected account endpoints obtain the account from the authenticated user rather than accepting an arbitrary account id.

## 12. How to run tests
```bash
python -m pytest -v
```

Tests use an in-memory SQLite database and do not modify the normal `bank.db`.

## 13. Known limitations
- SQLite does not provide true row-level `FOR UPDATE` locking like PostgreSQL.
- Tables are created with `create_all` rather than migrations.
- No refresh tokens, rate limiting, or pagination.
- One account per user.

## Example request bodies

Register:
```json
{
  "name": "John Doe",
  "email": "john@example.com",
  "password": "Password123"
}
```

Deposit:
```json
{
  "amount": 5000
}
```

Withdraw:
```json
{
  "amount": 1000
}
```

Transfer:
```json
{
  "receiverAccountId": 2,
  "amount": 1500
}
```
