# sori - Lending_solidity Security Audit

# Table of Contents

# Audit Overview

### Name

- sori - Lending_solidity Security Audit

### Git Repository

- [https://github.com/ExploitSori/Lending_solidity](https://github.com/ExploitSori/Lending_solidity)

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
| Critical | 1 | `Lending-sori-001` |
| High |  |  |
| Medium |  |  |
| Low |  |  |
| Informational |  |  |

# Findings

## Summary

| # | ID | Title | Severity |
| --- | --- | --- | --- |
| 1 | `Lending-sori-001` |  |  |

## #1 `Lending-sori-001` Overwriting deposit asset value in multiple deposits may lead to lost of user assets

### Description

- src/DreamAcademyLending.sol: 61, 67 [deposit()]

```jsx
	function deposit(address token_addr, uint256 amount) external payable{
		bool stat = chkTk(token_addr);
		uint256 val = msg.value;
		if(stat){
			//eth
			require(amount <= msg.value, "val error");
			deposit_list[msg.sender].eth = msg.value;
			deposit_list[msg.sender].eth_val = oracle.getPrice(address(0));
			deposit_list[msg.sender].eth_lblock = block.number;
		}
		else{
			usdc.transferFrom(msg.sender, address(this), amount);
			deposit_list[msg.sender].usdc = amount;
			deposit_list[msg.sender].usdc_val = oracle.getPrice(address(usdc));
			deposit_list[msg.sender].usdc_lblock = block.number;
			totalDeposit_usdc += amount;
		}
	}
```

유저가 ETH 또는 USDC를 deposit 하려고 하면, 본 컨트랙트는 유저가 입금한 금액을 더하는 것이 아니라 마지막에 호출한 값으로 변경한다. 만약 유저가 여러 번 deposit을 호출하여 입금하면, 이전에 입금한 값은 기록되지 않고 마지막에 입금한 금액만 인정된다.

### Impact

- Critical

유저 여러 번 입금하는 시나리오에서, 이전에 입금한 금액이 덮어씌워져 기록되지 않으므로, 유저는 자신의 자산 일부 또는 전체를 손실할 수 있다. 이로 인해 유저는 실질적으로 본인의 자산이 컨트랙트에 누적되지 않은 상태에서 자산을 잃게 된다.

### Recommendation

61, 67 번째 줄을 각각 `deposit_list[msg.sender].eth += msg.value;` , `deposit_list[msg.sender].usdc += amount;` 으로 수정하여 누적 합산 값을 저장한다.