# hakid29 - Lending_solidity Security Audit

# Table of Contents

# Audit Overview

### Name

- hakid29 - Lending_solidity Security Audit

### Git Repository

- [https://github.com/hakid29/Lending_solidity](https://github.com/hakid29/Lending_solidity)

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
| Critical | 1 | - `Lending-hakid29-001` |
| High |  |  |
| Medium |  |  |
| Low |  |  |
| Informational |  |  |

# Findings

## Summary

| # | ID | Title | Severity |
| --- | --- | --- | --- |
| 1 | `Lending-hakid29-001` | CIE pattern in withdraw() is vulnerable to re-entrancy attacks | Critical |
| 2 |  |  |  |

## #1 `Lending-hakid29-001` CIE pattern in withdraw() is vulnerable to re-entrancy attacks

### Description

- src/Lending.sol: 61-76 [withdraw()]

```jsx
 if (token == address(0)) { // Ether withdrawal
            uint256 collateralValueInUSDC = (user_.etherDeposit * priceOracle.getPrice(address(0))) / 1 ether;
            uint256 borrowedAmount = user_.usdcBorrow;
            require((collateralValueInUSDC - ((priceOracle.getPrice(address(0)) * amount) / 1 ether)) * 75 / 100 >= borrowedAmount, "Collateral is locked");

            payable(msg.sender).transfer(amount);
            user_.etherDeposit -= amount;
            Users[matchUserId[msg.sender]-1] = user_;
        } else if (token == usdc) { // USDC withdrawal
            require(user_.usdcDeposit >= amount, "Insufficient balance");

            ERC20(usdc).transfer(msg.sender, amount);
            totalUSDCDeposits -= amount;
            user_.usdcDeposit -= amount;
            Users[matchUserId[msg.sender]-1] = user_;
        } 
```

유저가 담보를 출금하기 위해 withdraw() 함수를 호출하면, 본 컨트랙트는 유저의 출금 가능한 금액이 출금 요청 금액보다 크거나 같은지 체크하고, 그 조건문을 통과하면 먼저 토큰 또는 이더를 transfer 한 후 스토리지 상의 변수를 업데이트한다. 이러한 Check-Interaction-Effect 패턴은 re-entrancy 공격에 취약하다.

### Impact

- Critical

재진입 공격에 성공하면 공격자는 사용자의 잔고가 줄어들기 전에 동일한 이더나 토큰을 여러 번 인출할 수 있으며, 이로 인해 전체 Vault의 자산이 고갈될 수 있다. 이러한 이유로 이 취약점은 매우 치명적이다.

### Recommendation

Re-entrancy 공격 시 피해를 방지하기 위해서는 자산 전송 전에 먼저 스토리지 상의 상태를 업데이트해야 한다. 이렇게 하면, re-entrancy를 시도할 때 이미 출금 요청 금액이 잔액에서 차감된 상태이므로 추가 출금 시도가 실패하게 된다.