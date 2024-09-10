# Observability

## Logging

### Don't expose sensetive data

App logs should not expose the following sensitive data:
* PII or Personally Identifiable Information such as username, first/last name, birth date, gender, billing/mailing address, email ID, contact number, credit card number, social security number, etc.
* Business names/contact information such as names of businesses; related persons including staff, clients, etc.; business relations; and business-related transactions with third parties.
* Security credentials, passwords, and auth tokens
* Financial data like card particulars, bank account numbers, and transaction amounts.