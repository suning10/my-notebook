## facade

- why
  - to hide steps of complex operation 
  - used for different layer of opeartions 

# When to Use the Facade Pattern

---

## What Problem Does It Actually Solve?

```
WITHOUT Facade — caller must know and orchestrate everything:

caller
  │
  ├──► knows Customer class
  ├──► knows SavingsAccount class
  ├──► knows Account registry format
  ├──► knows Customer ↔ Account linking
  ├──► knows validation rules
  ├──► knows notification system
  └──► knows audit logging

WITH Facade — caller knows ONE thing:

caller
  └──► bank.openSavingsAccount(customer, 2000)
                    │
                    └──► Facade handles ALL of the above internally
```

---

## The 7 Situations To Use It

---

### 1. Complex Subsystem With Many Steps

> **Use when:** A single logical operation requires coordinating many objects in a specific order.

```python
# ❌ WITHOUT Facade — 8 steps, caller must know all of them
def client_code_BAD():
    # Step 1: validate input
    if amount < SavingsAccount.MIN_BALANCE:
        raise ValueError("Too low")

    # Step 2: create account object
    acc = SavingsAccount(customer, amount)

    # Step 3: register in bank registry
    bank._accounts[acc.account_id] = acc

    # Step 4: link account to customer
    customer.add_account(acc)

    # Step 5: create welcome transaction
    txn = Transaction(TransactionType.DEPOSIT, amount)

    # Step 6: complete transaction
    txn.complete()

    # Step 7: notify customer
    customer.send_notification("Account opened!")

    # Step 8: audit log
    AuditLogger.log(f"Account {acc.account_number} created")

    return acc   # caller wrote 8 steps every time ❌
```

```python
# ✅ WITH Facade — 1 step, complexity hidden
def client_code_GOOD():
    acc = bank.open_savings_account(customer, amount)
    return acc   # done ✅
```

```java
// ❌ WITHOUT Facade — Java version of the mess
SavingsAccount acc = new SavingsAccount(
    customer, amount, SavingsAccount.INTEREST_RATE
);
if (amount < SavingsAccount.MIN_BALANCE)
    throw new IllegalArgumentException("Too low");
accounts.put(acc.getAccountId(), acc);
customer.addAccount(acc);
Transaction txn = new Transaction(
    TransactionType.DEPOSIT, amount, "", acc.getAccountId(), null
);
txn.complete();
customer.sendNotification("Account opened!");
AuditLogger.log("Account " + acc.getAccountNumber() + " created");

// ✅ WITH Facade
SavingsAccount acc = bank.openSavingsAccount(customer, amount);
```

---

### 2. Hiding a Poorly Designed or Legacy Subsystem

> **Use when:** The underlying system is messy, outdated, or hard to use directly — wrap it in a clean interface.

```python
# LEGACY system — ugly, inconsistent, hard to use
class LegacyPaymentProcessor:
    def init_txn_v2(self, acct_no, cur_code): ...
    def set_amt_pennies(self, pennies: int): ...
    def exec_debit_001(self, ref): ...
    def get_txn_status_code(self) -> int: ...  # 0=ok,1=fail,2=pending ?
    def close_txn_handle(self): ...

class LegacyFraudChecker:
    def run_check_type_A(self, acct): ...
    def run_check_type_B(self, acct): ...
    def get_risk_flag(self) -> str: ...  # "Y"/"N"/"M" ?

class LegacyNotificationService:
    def push_sms_v3(self, ph, msg, priority): ...


# ✅ FACADE wraps the ugly legacy into something clean
class PaymentFacade:
    def __init__(self):
        self._processor    = LegacyPaymentProcessor()
        self._fraud        = LegacyFraudChecker()
        self._notification = LegacyNotificationService()

    def process_payment(
        self,
        account_number: str,
        amount: float,
        phone: str
    ) -> bool:
        # Hide all legacy weirdness inside here
        self._fraud.run_check_type_A(account_number)
        self._fraud.run_check_type_B(account_number)
        if self._fraud.get_risk_flag() == "Y":
            return False

        self._processor.init_txn_v2(account_number, "USD")
        self._processor.set_amt_pennies(int(amount * 100))
        self._processor.exec_debit_001(ref="TXN-001")

        status = self._processor.get_txn_status_code()
        self._processor.close_txn_handle()

        if status == 0:
            self._notification.push_sms_v3(
                phone, "Payment successful", 1
            )
            return True
        return False


# Caller sees NONE of the legacy ugliness
facade = PaymentFacade()
success = facade.process_payment("123456789", 150.00, "+1555...")
```

---

### 3. Layered Architecture — Layer Boundary

> **Use when:** Separating layers (UI → Service → Domain → Data) — each layer boundary benefits from a Facade.

```
┌─────────────────────────────────────────────────────┐
│                  PRESENTATION LAYER                  │  ← React, CLI, Mobile
│              (knows only Service Facade)             │
└─────────────────────┬───────────────────────────────┘
                      │  calls
┌─────────────────────▼───────────────────────────────┐
│               SERVICE LAYER (FACADE)                 │  ← BankingService
│         BankingService.transferFunds(...)            │
│         BankingService.applyForLoan(...)             │
└──────┬──────────────┬──────────────┬────────────────┘
       │              │              │  coordinates
┌──────▼───┐   ┌──────▼───┐  ┌──────▼───┐
│ Account  │   │  Loan    │  │Notif.    │  ← Domain objects
│ Domain   │   │  Domain  │  │Service   │
└──────────┘   └──────────┘  └──────────┘
```

```python
# Service Layer Facade
class BankingService:
    """
    Facade for the entire banking domain.
    Controllers/UI only talk to this class.
    """
    def __init__(self):
        self._account_repo = AccountRepository()
        self._loan_repo    = LoanRepository()
        self._notifier     = NotificationService()
        self._auditor      = AuditService()
        self._risk         = RiskAssessmentService()

    # ── One facade method coordinates 4 domain services ──
    def transfer_funds(
        self,
        from_id: str,
        to_id: str,
        amount: float,
        initiated_by: str
    ) -> TransferResult:

        source = self._account_repo.find_by_id(from_id)
        target = self._account_repo.find_by_id(to_id)

        # Domain service 1: risk check
        self._risk.assess_transfer(source, amount)

        # Domain service 2: execute transfer
        txn = source.transfer(target, amount)

        # Domain service 3: persist
        self._account_repo.save(source)
        self._account_repo.save(target)

        # Domain service 4: notify both parties
        self._notifier.notify_transfer(source.customer, txn)
        self._notifier.notify_received(target.customer, txn)

        # Domain service 5: audit trail
        self._auditor.log_transfer(txn, initiated_by)

        return TransferResult(success=True, txn=txn)


# ── Controller/UI only needs this ──────────────────
class TransferController:
    def __init__(self, service: BankingService):
        self._service = service

    def handle_transfer_request(self, request):
        result = self._service.transfer_funds(   # ← one call
            request.from_id,
            request.to_id,
            request.amount,
            request.user_id
        )
        return {"success": result.success}
```

---

### 4. Third-Party Library Integration

> **Use when:** Wrapping an external library so your app never directly depends on it — swap the library without changing your code.

```python
# ❌ WITHOUT Facade — your app is tightly coupled to Stripe
import stripe

class PaymentController:
    def charge(self, amount, token):
        stripe.api_key = "sk_live_..."          # Stripe detail
        intent = stripe.PaymentIntent.create(   # Stripe detail
            amount=int(amount * 100),           # Stripe detail
            currency="usd",                     # Stripe detail
            payment_method=token,               # Stripe detail
            confirm=True                        # Stripe detail
        )
        return intent.status == "succeeded"     # Stripe detail
        # Switching to PayPal = rewrite ALL callers ❌


# ✅ WITH Facade — your app depends only on PaymentGateway
class PaymentGateway(ABC):
    @abstractmethod
    def charge(self, amount: float, token: str) -> bool:
        pass

    @abstractmethod
    def refund(self, transaction_id: str) -> bool:
        pass


class StripeFacade(PaymentGateway):
    def __init__(self, api_key: str):
        import stripe
        stripe.api_key = api_key
        self._stripe = stripe

    def charge(self, amount: float, token: str) -> bool:
        intent = self._stripe.PaymentIntent.create(
            amount=int(amount * 100),
            currency="usd",
            payment_method=token,
            confirm=True
        )
        return intent.status == "succeeded"

    def refund(self, transaction_id: str) -> bool:
        refund = self._stripe.Refund.create(
            payment_intent=transaction_id
        )
        return refund.status == "succeeded"


class PayPalFacade(PaymentGateway):
    def charge(self, amount: float, token: str) -> bool:
        # PayPal-specific implementation
        ...

    def refund(self, transaction_id: str) -> bool:
        # PayPal-specific implementation
        ...


# ── App only knows PaymentGateway interface ────────
class BankPaymentService:
    def __init__(self, gateway: PaymentGateway):
        self._gateway = gateway   # inject whichever

    def process(self, amount, token):
        return self._gateway.charge(amount, token)
        # Switching Stripe → PayPal = change ONE injection ✅

# Stripe
service = BankPaymentService(StripeFacade("sk_live_..."))

# Switching to PayPal — zero change in BankPaymentService
service = BankPaymentService(PayPalFacade())
```

---

### 5. Reducing Cognitive Load For a Team

> **Use when:** Junior devs or other teams consume your module — shield them from internals.

```python
# ❌ WITHOUT Facade — new dev must understand 6 classes to do anything
#
# "To send a wire transfer you need to:
#  1. Instantiate WireTransferRequest
#  2. Call ComplianceChecker.validate()
#  3. Call FXRateService.convert() if cross-currency
#  4. Call CorrespondentBankRouter.route()
#  5. Call SwiftMessageBuilder.build()
#  6. Call TransactionLedger.record()"
#
# Every team member repeats this. One mistake = bug.


# ✅ WITH Facade — one method, safe for everyone
class WireTransferFacade:
    """
    Public API for wire transfers.
    Other teams call ONLY this class.
    """

    def send_domestic(
        self,
        from_account: str,
        to_account: str,
        amount: float,
        reference: str
    ) -> WireResult:
        """Send a domestic wire. That's all you need to know."""
        # All the complexity is buried here
        req       = WireTransferRequest(from_account, to_account, amount)
        validated = self._compliance.validate(req)
        routed    = self._router.route_domestic(validated)
        msg       = self._swift.build(routed)
        self._ledger.record(msg)
        return WireResult(success=True, ref=reference)

    def send_international(
        self,
        from_account: str,
        to_account: str,
        amount: float,
        target_currency: str,
        reference: str
    ) -> WireResult:
        """Send an international wire. That's all you need to know."""
        req       = WireTransferRequest(from_account, to_account, amount)
        validated = self._compliance.validate(req)
        converted = self._fx.convert(amount, "USD", target_currency)
        routed    = self._router.route_international(validated)
        msg       = self._swift.build_international(routed, converted)
        self._ledger.record(msg)
        return WireResult(success=True, ref=reference)
```

---

### 6. Unit Testing — Easier Mocking

> **Use when:** You want to test business logic without spinning up real subsystems.

```python
# ── Real facade ───────────────────────────────────────
class BankingService:
    def __init__(
        self,
        account_repo,
        notifier,
        auditor
    ):
        self._account_repo = account_repo
        self._notifier     = notifier
        self._auditor      = auditor

    def close_account(self, account_id: str) -> bool:
        account = self._account_repo.find_by_id(account_id)
        if account.balance != 0:
            return False
        account.close()
        self._account_repo.save(account)
        self._notifier.send(account.customer, "Account closed")
        self._auditor.log(f"Closed {account_id}")
        return True


# ── Test — mock only the facade's dependencies ────────
from unittest.mock import MagicMock

def test_close_account_with_zero_balance():
    # Arrange
    mock_repo     = MagicMock()
    mock_notifier = MagicMock()
    mock_auditor  = MagicMock()

    mock_account         = MagicMock()
    mock_account.balance = 0.0
    mock_repo.find_by_id.return_value = mock_account

    service = BankingService(
        mock_repo, mock_notifier, mock_auditor
    )

    # Act
    result = service.close_account("ACC-001")

    # Assert
    assert result == True
    mock_account.close.assert_called_once()         # ✅
    mock_notifier.send.assert_called_once()         # ✅
    mock_auditor.log.assert_called_once()           # ✅
    # No real DB, no real email, no real audit log needed
```

---

## When NOT to Use It

```python
# ❌ DON'T use Facade for simple operations
class OverEngineeredFacade:
    def get_customer_name(self, customer: Customer) -> str:
        return customer.full_name   # Facade adds zero value here


# ❌ DON'T use Facade as a dumping ground (God Object)
class BadFacade:
    def process_loan(self): ...
    def calculate_tax(self): ...      # unrelated
    def generate_report(self): ...    # unrelated
    def send_marketing_email(self):...# unrelated
    def manage_employees(self): ...   # unrelated
    # This is a God Class, not a Facade ❌


# ❌ DON'T use Facade to hide design flaws forever
class HidingTheProblem:
    def do_thing(self):
        # 500 lines hiding a fundamentally broken subsystem
        # Fix the subsystem instead ❌


# ✅ DO use Facade when:
#   - Multiple steps must be orchestrated in order
#   - Subsystem is legitimately complex (not just broken)
#   - Multiple callers repeat the same orchestration
#   - You want a clean public API over internal complexity
```

---

## Decision Flowchart

```
Does the operation require coordinating
multiple objects/services?
         │
         ▼
        YES
         │
         ▼
Do multiple callers repeat
the same orchestration steps?
         │
         ├── YES ──► Use Facade ✅
         │
         └── NO ───► Maybe just a helper method
                     in the calling class

Is an external library or legacy
system hard to use directly?
         │
         ├── YES ──► Use Facade as a wrapper ✅
         │
         └── NO ───► Use it directly

Do you want to enforce a
clean public API boundary?
(e.g., other teams, microservices)
         │
         ├── YES ──► Use Facade ✅
         │
         └── NO ───► Facade may be overkill
```

---

## Summary Table

```
┌─────────────────────────────────┬──────────┬──────────────────────────────┐
│ Situation                       │ Use?     │ Why                          │
├─────────────────────────────────┼──────────┼──────────────────────────────┤
│ Multi-step orchestration        │ ✅ YES   │ Hides coordination logic     │
│ Legacy/3rd party wrapping       │ ✅ YES   │ Isolates dependency          │
│ Layer boundaries                │ ✅ YES   │ Clean separation of concerns │
│ Team API boundary               │ ✅ YES   │ Reduces cognitive load       │
│ Easier testing                  │ ✅ YES   │ Mock one class not many      │
│ Simple single-object operations │ ❌ NO    │ Adds complexity for no gain  │
│ Hiding fundamentally bad design │ ❌ NO    │ Fix the design instead       │
│ Catch-all utility class         │ ❌ NO    │ Becomes a God Object         │
└─────────────────────────────────┴──────────┴──────────────────────────────┘
```