# Basic Concept

## Types of Accounts

There are two types of accounts:

+ Accounts whose balance at a point in time is meaningful are called *balance sheet
accounts*. There are two types of such accounts: "**Assets**" and "**Liabilities**".
+ The other accounts, that is, those whose balance is not particularly meaningful but for
which we are interested in calculating changes over a period of time are called *income
statement accounts*. Again, there are two kinds: "**Income**" and "**Expenses**".

The next thing is we need to consider the *usual sign of an account's balance*. The great
*majority* of accounts in the double-entry system tend to have a balance with always a
*positive* sign, or always a *negative* sign:

+ For a balance sheet accounts, Assets normally have positive balances, and Liabilities
normally have negative balances.
+ For income statement accounts, Expenses normally have a positive balance, and Income
accounts normally have a negative balance.

To be more specific:

+ **Assets (+)**. Asset accounts represent *something the owner has*.
  + Banking accounts.
  + Cash account, which counts how much money is in your wallet.
  + Investments (their units aren't dollars in this case, but rather some number of shares
  of some mutual fund or stock).
  + House
+ **Liabilities (-)**. A liability account represents *something the owner owes*
  + Credit card
  + A loan
+ **Expenses (+)**. An expense account represents *something you've received*, perhaps by
exchanging something else to purchase it. This type of account will seem pretty natural:
food, drinks and most other categories of things you typically spend your disposable
income on. However, taxes are also typically tracked by an expense account.
+ **Income (-)**. An income account is used to count *something you've given away* in
order to receive something else. For most people with jobs, that is the value of their
time (a salary income). Specifically, here we’re talking about the *gross* income.

In Beancount, all account names, without exception, must be associated to one of the types
of accounts described above. Since the type of an account never changes during its
lifetime, we will make its type a part of an account's name, as a *prefix*:

+ The qualified account name for restaurant will be `Expenses:Restaurant`.
+ For the bank checking account, the qualified account name will be `Assets:Checking`.

## Trial Balance

We can easily reorder all the accounts such that all the *Asset* accounts appear together
at the top, then all the *Liabilities* accounts, then *Income*, and finally *Expenses*
accounts. We are simply changing the order without modifying the structure of
transactions, in order to group each type of accounts together.

If we sum up the postings on all of the accounts and render just the account name and its
final balance on the right, we obtain a report we call the "trial balance".

## Income Statement

One kind of common information that is useful to extract from the list of transactions is
a summary of changes in *Income* statement accounts during a particular period of time.
This tells us how much money was earned and spent during this period, and the difference
tells us how much profit(or loss) was incurred. We call this the "*net income*".

## Clearing Income

Sometimes, we want to zero out the balances of the *Income* and *Expenses* accounts.
Beancount calls this basic transformation "*clearing*". It is carried out by:

1. Computing the balances of those accounts from the beginning of time to the start of the
reporting period.
2. Inserting transactions to empty those balances and transfer them to some other account
that isn't *Income* nor *Expenses*. For example, if the restaurant expense account for
those 16 years amounts to `$85,321` on Jan 1, 2016, it would insert a transaction of
`$-85,321` to restaurants and `$85,321` to "previous earnings".

## Equity

The account that receives those previously accumulated incomes is called "*Previous
Earnings*". It lives in a fifth and final type of accounts: **Equity**. They are most
often used to transfer amounts to build up reports, and the owner usually doesn't post
changes to those types of accounts; *the software does that automatically* when clearing
net income.

The account type "equity" is used for accounts that hold a summary of the net income
implied by all the past activity. There are a few different *Equity* accounts in use in
Beancount:

+ **Previous Earnings** or **Retained Earnings**. An account used to hold the sum total of
*Income* and *Expenses* balances from the beginning of time until the *beginning* of a
reporting period.
+ **Current Earnings**, also called *Net Income*. An account used to contain the sum of
*Income* and *Expenses* balances incurred during the reporting period.
+ **Opening Balances**. An equity account used to counterbalance deposits used to
initialize accounts. This type of account is used when we truncate the past history of
transactions.

## Balance Sheet

Another kind of summary is a listing of the owner's assets and debts, for each of the
accounts. This answers the question: "*Where's the money?*" And "*Once all debts are paid
off, how much are we left with?*" This is called **net worth**.

If the *Income* and *Expenses* accounts have been cleared to zero and all their balances
have been transferred to Equity accounts, the net worth should be equal to the sum of all
the Equity accounts.

## Accounting Equations

+ $A$: the sum of all *Assets* postings
+ $L$: the sum of all *Liabilities* postings
+ $X$: the sum of all *Expenses* postings
+ $I$: the sum of all *Income* postings
+ $E$: the sum of all *Equity* postings

We can say that

$$
A + L + E + X + I = 0
$$

Moreover, the sum of postings from *Income* and *Expenses* is the *Net Income*, we call
this $NI$. If we adjust the equity to reflect the total *Net Income* effect by clearing
the income to the Equity retained earnings account, we get an updated Equity value $E'$.

$$
E' = E + NI = E + X + I
$$

And we have a simplified accounting equation:

$$
A + L + E' = 0
$$

If we were to adjust the signs for credits and debits, we have the rule from accounting:
**Assets equals Liabilities plus Equity**.
