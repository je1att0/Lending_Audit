# castle - Lending_solidity Security Audit

# Table of Contents

# Audit Overview

### Name

- castle - Lending_solidity Security Audit

### Git Repository

- [https://github.com/rivercastleone/Lending_solidity](https://github.com/rivercastleone/Lending_solidity)

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
| Critical | 1 | - `Lending-castle-001` |
| High |  |  |
| Medium |  |  |
| Low |  |  |
| Informational |  |  |

# Findings

## Summary

| # | ID | Title | Severity |
| --- | --- | --- | --- |
| 1 | `Lending-castle-001` | NotOwner calling initializeLendingProtocol() may lead to theft of all the tokens in the pool | Critical |

## #1 `Lending-castle-001` NotOwner calling initializeLendingProtocol() may lead to theft of all the tokens in the pool

### Description

- src/Lending.sol: 64-67 [initializeLendingProtocol()]

```jsx
function initializeLendingProtocol(address addr) external payable {
        usdc = IERC20(addr);
        usdc.transferFrom(msg.sender, address(this), msg.value);
    }
```

본 컨트랙트에서는 initializeLendingProtocol 함수를 아무나 호출할 수 있다. 만약 공격자가 자신이 발행한 가짜 ERC-20 토큰을 usdc 주소로 설정한 뒤, 해당 토큰을 이용해 예치하거나 대출을 받을 수 있다면, 실제 담보 없이 프로토콜의 자산을 빼돌릴 수 있게 된다. 이로 인해 프로토콜에 심각한 손실이 발생할 수 있다.

### Impact

- Critical

이 공격이 성공할 경우, 전체 프로토콜의 자산이 손실될 가능성이 매우 크며, 신뢰성 또한 심각하게 훼손될 수 있다.

### Recommendation

initializeLendingProtocol 함수는 프로토콜의 소유자나 관리자만 호출할 수 있도록 권한 제어를 추가해야 한다. 이를 위해 onlyOwner와 같은 접근 제어 제도를 사용할 수 있다. 이는 OpenZeppelin의 Ownable 계약을 상속받아 해결할 수 있다.