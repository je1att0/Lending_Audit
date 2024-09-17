# jeremy - Lending_solidity Security Audit

# Table of Contents

# Audit Overview

### Name

- jeremy - Lending_solidity Security Audit

### Git Repository

- [https://github.com/WOOSIK-jeremy/Lending_solidity](https://github.com/WOOSIK-jeremy/Lending_solidity)

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
| Critical | 1 | - `Lending-jeremy-001` |
| High |  |  |
| Medium |  |  |
| Low |  |  |
| Informational |  |  |

# Findings

## Summary

| # | ID | Title | Severity |
| --- | --- | --- | --- |
| 1 | `Lending-jeremy-001` | Not checking whether depositing token is valid token may lead to theft of all the tokens in the pool | Critical |

## #1 `Lending-jeremy-001` Not checking whether depositing token is valid token may lead to theft of all the tokens in the pool

### Description

- src/Lending.sol: 30-42 [deposit()]

```jsx
function deposit(address token, uint256 tokenAmount) public payable {
        if(token == address(0x00)) {
            require(msg.value >= tokenAmount, "your ether lower than deposit ether");
            accounts[msg.sender] += tokenAmount;
            firstAccount[msg.sender] += tokenAmount;
        }
        else {
            uint256 allow = usdc.allowance(msg.sender, address(this));
            require(allow >= tokenAmount, "your token lower than deposit token");
            usdc.transferFrom(msg.sender, address(this), tokenAmount);
            tokenAccount[msg.sender] += tokenAmount;
        }
    }
```

본 컨트랙트에서는 deposit한 토큰이 실제로 usdc인지 체크하는 로직이 존재하지 않는다. 이럴 경우, 공격자는 임의의 토큰을 발행하여 해당 토큰을 deposit해 담보가 있는 것처럼 위장한 후 풀에 있는 금액을 빼내올 수 있다.

### Impact

- Critical

공격자는 가짜 토큰을 통해 무제한으로 실제 자산을 인출할 수 있으며, 프로토콜 내 유동성 자산이 고갈될 가능성이 크다. 이는 시스템 전반에 걸친 파괴적인 영향을 초래할 수 있다.

### Recommendation

deposit 함수에서 전달된 token 주소가 실제로 USDC인지 확인하는 로직을 추가해야 한다. 이를 통해 USDC 외의 다른 토큰은 예치할 수 없도록 제한해야 한다.