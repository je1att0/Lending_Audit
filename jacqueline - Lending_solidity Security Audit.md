# jacqueline - Lending_solidity Security Audit

# Table of Contents

# Audit Overview

### Name

- jacqueline - Lending_solidity Security Audit

### Git Repository

- [https://github.com/je1att0/Lending_solidity](https://github.com/je1att0/Lending_solidity)

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
| Critical | 1 | - `Lending-jacqueline-001` |
| High |  |  |
| Medium |  |  |
| Low |  |  |
| Informational |  |  |

# Findings

## Summary

| # | ID | Title | Severity |
| --- | --- | --- | --- |
| 1 | `Lending-jacqueline-001` | Liquidator not sending collateral for liquidated assets may lead to collateral shortfall | Critical |

## #1 `Lending-jacqueline-001` Liquidator not sending collateral for liquidated assets may lead to collateral shortfall

### Description

- src/Lending.sol: 230-242 [liquidate()]

```jsx
    function liquidate (address _userAddress, address _tokenAddress, uint256 _amount) public payable {
        LoanAccount memory account = updatedAccount(_userAddress);
        uint ETHprice = upsideOracle.getPrice(address(0x0));
        require(account.liquidable == true && account.limitLeft == 0, "Loan is not undercollateralized"); 
        require(account.liquidableLeft>=_amount, "Exceed amount to liquidate");
        
        uint liquidate_amount = (_amount/ETHprice)*10**18;
        account.depositedETH -= liquidate_amount;
        account.liquidableLeft -= _amount;
        
        payable(msg.sender).transfer(liquidate_amount);
        accounts[_userAddress] = account;
    }
```

본 컨트랙트에서는 liquidate 함수의 호출자가 _userAddress를 청산시킬 때, 청산 대가로 _userAddress의 담보를 가져가는데, 그에 해당하는 token을 담보로 맡기는 부분은 존재하지 않는다. 이로 인해 공격자는 담보가 충분히 확보되지 않았음에도 청산을 실행할 수 있으며, 사용자가 충분한 자산을 잃는 상황이 발생할 수 있다. 공격자는 의도적으로 청산 한도를 넘는 양의 ETH를 청산하려 시도할 수도 있고, 청산자가 ETH를 가져가는 동안 담보로 맡길 자산을 제공하지 않으므로 Vault에 충분한 자산이 보장되지 않아 시스템이 불안정해질 수 있다.

### Impact

- Critical

청산 과정에서 담보 교환이 제대로 이루어지지 않으면, 프로토콜 내에서 담보 부족 사태가 발생할 수 있으며, 이는 플랫폼의 전체 유동성에 큰 영향을 미친다. 결과적으로 시스템의 안정성과 유동성을 보장하는 데 실패할 가능성이 있으며, 이는 악용되었을 때 큰 손실로 이어질 수 있다.

### Recommendation

청산자가 _userAddress의 ETH 담보를 가져갈 때, 그에 상응하는 양의 **토큰**을 Vault에 입금하도록 강제해야 한다. 이를 위해서는 _tokenAddress와 _amount를 기반으로 청산자가 해당 토큰을 제공했는지 확인하는 과정이 필요하다.