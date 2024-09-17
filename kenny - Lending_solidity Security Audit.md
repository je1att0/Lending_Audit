# kenny - Lending_solidity Security Audit

# Table of Contents

# Audit Overview

### Name

- kenny - Lending_solidity Security Audit

### Git Repository

- [https://github.com/55hnnn/Lending_solidity](https://github.com/55hnnn/Lending_solidity)

### Severity Categories

- Critical
    
    The attack cost is low (not requiring much time or effort to succeed in the
    actual attack), and the vulnerability causes a high-impact issue. (e.g., Effect on
    service availability, Attacker taking financial gain)
    
- High
    
    An attacker can succeed in an attack which clearly causes problems in the
    service’s operation. Even when the attack cost is high, the severity of the issue
    is considered “high” if the impact of the attack is remarkably high.
    
- Medium
    
    An attacker may perform an unintended action in the service, and the action
    may impact service operation. However, there are some restrictions for the
    actual attack to succeed.
    
- Low
    
    An attacker can perform an unintended action in the service, but the action
    does not cause significant impact or the success rate of the attack is
    remarkably low.
    
- Informational
    
    Any informational findings that do not directly impact the user or the protocol.
    

### Finding Breakdown by Severity

| Category | Count | Findings |
| --- | --- | --- |
| Critical | 1 | - `Lending-kenny-001` |
| High |  |  |
| Medium |  |  |
| Low |  |  |
| Informational |  |  |

# Findings

## Summary

| # | ID | Title | Severity |
| --- | --- | --- | --- |
| 1 | `Lending-kenny-001` |  Not checking whether repaying token is same as borrowed token may lead to theft of all the tokens in the pool | Critical |

## #1 `Lending-kenny-001` Not checking whether repaying token is same as borrowed token may lead to theft of all the tokens in the pool

### Description

- src/Lending.sol: 77-90 [repay()]

```jsx
    function repay(address token, uint256 amount) external {
        require(token != address(0), "Invalid token address");
        require(amount > 0, "Amount must be greater than zero");
        _updateLoanValue(token); // 이자율 반영

        uint256 debt = debts[msg.sender][token];
        require(debt > 0, "No outstanding debt");
        require(amount <= debt, "Repayment amount exceeds debt");

        ERC20(token).transferFrom(msg.sender, address(this), amount);
        debts[msg.sender][token] -= amount;
        
        totalDebt -= amount;
    }
```

본 컨트랙트에서는 유저가 상환하려는 토큰이 대출했던 토큰의 주소와 일치하는지 체크하지 않는다. 공격자는 이 점을 이용하여 임의의 토큰을 발행해 그 토큰으로 상환해서 실제로 대출했던 토큰을 상환하지 않았지만 상환한 것처럼 할 수 있다. 

### Impact

- Critical

본 취약점을 이용해서 공격자는 해당 풀에 있는 모든 금액을 대출한 후, 임의의 토큰으로 상환하여 모든 금액을 빼돌릴 수 있다.

### Recommendation

상환하려는 토큰이 빌린 토큰과 일치하는지 체크하는 로직을 추가한다.