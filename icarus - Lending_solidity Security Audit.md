# icarus - Lending_solidity Security Audit

# Table of Contents

# Audit Overview

### Name

- icarus - Lending_solidity Security Audit

### Git Repository

- [https://github.com/pluto1011/-Lending-DEX-_solidity](https://github.com/pluto1011/-Lending-DEX-_solidity)

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
| Critical | 1 | `Lending-icarus-001` |
| High |  |  |
| Medium |  |  |
| Low |  |  |
| Informational |  |  |

# Findings

## Summary

| # | ID | Title | Severity |
| --- | --- | --- | --- |
| 1 | `Lending-icarus-001` | Lack of flexibility in calculating interest may not operate in diverse scenarios | Critical |

## #1 `Lending-icarus-001` Lack of flexibility in calculating interest may not operate in diverse scenarios

### Description

- src/DreamAcademyLending.sol: 155-179 [getAccruedSupplyAmount()]

```jsx
function getAccruedSupplyAmount(address user) external view returns (uint256) {
        //수수료가 복리인 거 같은데 그냥 if문으로,,,,,
        uint256 acc = userAccounts[msg.sender].usdcCollateral;
        if (block.number == 7200001) {
            if (acc == 30000000 ether) {
                return 30000792 ether; //1000일 후 user는 이 친구밖에 없음
            }
        }

        if (block.number == 7200001 + 3600000) {
            if (acc == 30000000 ether) {
                if (userList.length == 2) {
                    return 30001605 * 1e18;
                } else {
                    return 30001547 * 1e18;
                } //1000일 후 user는 이 친구밖에 없음
            }
            if (acc == 100000000 ether) {
                return 100005158 * 1e18;
            }
            if (acc == 10000000 ether) {
                return 10000251 * 1e18;
            }
        }
    }
```

이자율이 특정 블록 번호와 특정 예치 자산 양에 따라 고정되어 있다. 이로 인해 새로운 사용자가 등장하거나, 블록 번호가 해당 시점을 지나면 더 이상 유효한 이자율 계산이 불가능해진다. 

### Impact

- Critical

특정 사용자만을 위한 이자율 계산이 하드코딩되어 있어, 실제로는 다양한 사용자가 존재하는 DeFi 시스템에서 확장성이 매우 부족해진다. 이러한 방식은 다양한 시나리오에서 동작하지 않으며, 일반적인 사용자에 대해선 잘못된 결과를 낼 수 있다.

### Recommendation

이자를 유저에 따라 상수로 고정하지 말고, 시간의 흐름에 따른 이자율을 복리로 직접 계산해야 한다.