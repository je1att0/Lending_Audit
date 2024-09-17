# entropy - Lending_solidity Security Audit

# Table of Contents

# Audit Overview

### Name

- entropy - Lending_solidity Security Audit

### Git Repository

- [https://github.com/Entropy1110/Lending_solidity](https://github.com/Entropy1110/Lending_solidity)

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
| Critical | 1 | - `Lending-entropy-001` |
| High |  |  |
| Medium |  |  |
| Low |  |  |
| Informational |  |  |

# Findings

## Summary

| # | ID | Title | Severity |
| --- | --- | --- | --- |
| 1 | `Lending-entropy-001` | **Lack of liquidator incentives may lead to inefficient liquidations** | High |

## #1 `Lending-entropy-001` **Lack of liquidator incentives may lead to inefficient liquidations**

### Description

- src/Lending.sol: 144-167 [liquidate()]

설명

```jsx
 function liquidate(address user, address token, uint256 _amount) external {
        uint256 borrowed = userBalances[user].borrowedAsset;
        require(borrowed > 0, "No debt to liquidate");

        uint256 ethCollateral = userBalances[user].depositedETH;
        uint256 ethPrice = priceOracle.getPrice(address(0));
        uint256 assetPrice = priceOracle.getPrice(token);
        uint256 collateralValue = ethCollateral * ethPrice;
        uint256 debtValue = borrowed * assetPrice;

        require(collateralValue * 75 / 100 < debtValue, "Not liquidatable"); // LT = 75% 
        
        if (_amount == borrowed){
            require(borrowed <= 100, "can liquidate the whole position when the borrowed amount is less than 100");
        }else{
            require(borrowed * 25 / 100 >= _amount, "can liquidate 25% of the borrowed amount");
        }

        userBalances[user].borrowedAsset -= _amount;
        totalBorrowedUSDC -= _amount;
        userBalances[user].depositedETH -= _amount * assetPrice / ethPrice;

        require(ERC20(token).transferFrom(msg.sender, address(this), _amount), "Liquidation transfer failed");
    }
```

본 컨트랙트에서는 청산자가 liquidate 함수를 호출하여 user을 청산시켜도 그에 대한 보상을 받는 코드가 존재하지 않는다. 이럴 경우, 청산자가 사용자의 채무를 상환하더라도 그에 대한 보상을 받지 못하므로, 청산 과정에서 손해를 보게 된다. 이는 청산자가 청산을 꺼리게 만드는 구조이다. 청산 유인이 없을 경우, 포지션이 담보 부족 상태임에도 청산이 이루어지지 않아 프로토콜 전체가 부실해질 가능성이 있다.

### Impact

- High

청산 메커니즘이 제대로 작동하지 않으면, 담보 부족 상태인 사용자들의 대출이 계속 유지되며, 장기적으로는 프로토콜의 유동성 문제를 초래할 수 있다. 이는 시스템 전체의 안정성에 부정적인 영향을 미칠 수 있다.

### Recommendation

청산자가 사용자의 포지션을 청산할 때, 해당 청산자가 일정 비율의 담보를 보상으로 받는 코드를 추가해야 한다.