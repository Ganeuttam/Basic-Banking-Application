# Run this project in VS Code on Windows

## 1. Open the project
Extract the ZIP and open the `basic-banking-api` folder in VS Code.

## 2. Open Terminal
VS Code -> Terminal -> New Terminal.

## 3. Create virtual environment
```powershell
python -m venv venv
```

## 4. Activate it
```powershell
venv\Scripts\activate
```

If PowerShell blocks activation:
```powershell
Set-ExecutionPolicy -Scope CurrentUser RemoteSigned
```
Then activate again.

## 5. Install packages
```powershell
python -m pip install --upgrade pip
pip install -r requirements.txt
```

## 6. Create .env
```powershell
copy .env.example .env
```

You can replace `SECRET_KEY` in `.env` with:
```powershell
python -c "import secrets; print(secrets.token_hex(32))"
```

## 7. Run tests
```powershell
python -m pytest -v
```

## 8. Start the API
```powershell
uvicorn app.main:app --reload
```

## 9. Open Swagger
Open:
http://127.0.0.1:8000/docs

## 10. Test the normal banking flow
1. Register user 1.
2. Register user 2.
3. Login as user 1.
4. Click **Authorize** and paste only the access token.
5. Open `/api/accounts/me` and note user 1's `accountId`.
6. Login as user 2 separately and note user 2's `accountId`.
7. Login back as user 1 and authorize user 1's token.
8. Deposit 5000.
9. Withdraw 1000.
10. Transfer 1500 to user 2's account id.
11. Check transactions.
12. Logout and confirm that the old token receives 401.

## Important
Do not upload `.env`, `venv`, `bank.db`, or `__pycache__` to GitHub.
