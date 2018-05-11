# Making adjustments to an already completed transaction

## Changing the Respondent from a Credit Trade from one organization to another
### Below was an actual script we used to make adjustments to the server

Get some of the details for the credit transaction
```
select * from credit_trade 
where id = 52;
```

Let's say we got 48 from the previous statement
Verify which organization this is:
```
select * from organization 
where id = 48; // Park Land Fuel
```

Assuming we know which organization we're changing the respondent to.
Verify the destination ID
```
select * from organization 
where id = 33; // Parkland Refining
```

So let's actually update the credit trade so it points to the right respondent
```
update credit_trade 
set respondent_id = 33 
where id = 52 and respondent_id = 48;
```

Next, we make sure to update the history of the credit trade
```
update credit_trade_history 
set respondent_id = 33 
where credit_trade_id = 52 and respondent_id = 48;
```

We should update the balance so it points to the right respondent
```
update organization_balance 
set organization_id = 33 
where credit_trade_id = 52 and organization_id = 48;
```

We should update the balance of the previous respondent (reverting to the last amount before the change was made)
2211 is an example value based on credit_trade
```
update organization_balance a 
set validated_credits = (
  select validated_credits - 2211 
  from organization_balance 
  where 
    organization_id = a.organization_id and 
    credit_trade_id = a.credit_trade_id
) where credit_trade_id = 209 and organization_id = 48; // 16100 
```

Next, we update the balance of the respondent we just updated the credit trade to.
266172, in this example, is the last balance before this transaction was applied.
So we should update the balance we just recently modified in (update organization_balance set organization_id = 33 where credit_trade_id = 52 and organization_id = 48;)

```
update organization_balance a 
set validated_credits = 266172 
where organization_id = 33 and credit_trade_id = 52; // 266172
```

Now we have to update all the succeeding balances
```
update organization_balance a 
set validated_credits = (
  select validated_credits + 2211 
  from organization_balance 
  where organization_id = a.organization_id and 
    credit_trade_id = a.credit_trade_id
) where organization_id = 33 and credit_trade_id = 55; // 302017
```
```
update organization_balance a 
set validated_credits = (
  select validated_credits + 2211 
  from organization_balance 
  where organization_id = a.organization_id and 
    credit_trade_id = a.credit_trade_id
) where organization_id = 33 and credit_trade_id = 76; // 299854
```
```
update organization_balance a 
set validated_credits = (
  select validated_credits + 2211 
  from organization_balance 
  where organization_id = a.organization_id and 
    credit_trade_id = a.credit_trade_id
) where organization_id = 33 and credit_trade_id = 161; // 299714
```
```
update organization_balance a 
set validated_credits = (
  select validated_credits + 2211 
  from organization_balance 
  where organization_id = a.organization_id and 
  credit_trade_id = a.credit_trade_id
) where organization_id = 33 and credit_trade_id = 196; // 303325
```
```
update organization_balance a 
set validated_credits = (
  select validated_credits + 2211 
  from organization_balance 
  where organization_id = a.organization_id and 
  credit_trade_id = a.credit_trade_id
) where organization_id = 33 and credit_trade_id = 198;// 323325
```
```
update organization_balance a 
set validated_credits = (
  select validated_credits + 2211 
  from organization_balance 
  where organization_id = a.organization_id and 
    credit_trade_id = a.credit_trade_id
) where organization_id = 33 and credit_trade_id = 209;// 307225
```

## Changing the number of credits in a credit trade

So in the example below, we'll be modifyin from 10833 to 10883 for a specific credit trade
```
update credit_trade 
set number_of_credits = 10883 
where id = 68 and respondent_id = 10;
```

We should update the history
```
update credit_trade_history 
set number_of_credits = 10883 
where credit_trade_id = 68 and respondent_id = 10;
```

Let's do a quick glance on how the organization_balance looks like
```
select * 
from organization_balance 
where organization_id = 10;
```

At this point we know that there's a difference of 50 from the previous credit trade
So let's add 50 to each of the organization balance from when the balance should occur
```
update organization_balance a 
set validated_credits = (
  select validated_credits + 50 
  from organization_balance 
  where organization_id = a.organization_id and 
    credit_trade_id = a.credit_trade_id
) where organization_id = 10 and credit_trade_id = 68; // 10883
```
```
update organization_balance a 
set validated_credits = (
  select validated_credits + 50 
  from organization_balance 
  where organization_id = a.organization_id and 
    credit_trade_id = a.credit_trade_id
) where organization_id = 10 and credit_trade_id = 69; // 18023
```
```
update organization_balance a 
set validated_credits = (
  select validated_credits + 50 
  from organization_balance 
  where organization_id = a.organization_id and 
    credit_trade_id = a.credit_trade_id
) where organization_id = 10 and credit_trade_id = 94; // 24801
```
