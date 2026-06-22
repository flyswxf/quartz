```maude
mod SIMPLE-BANK is
	sorts Account .
	op acc : Nat Nat -> Account .
	vars ID1 ID2 AMT BAL1 BAL2 : NAT .
	crl [transfer] :
		acc(ID1, BAL1) acc(ID2, BAL2) => acc(ID1, BAL1-AMT) acc(ID2, BAL2+AMT)
	if AMT < BAL1 /\ AMT > 0 .
	crl [withdraw] :
		acc(ID1, BAL1) => acc(ID1, BAL1 - AMT)
	if AMT < BAL1 /\ AMT > 0 .
```