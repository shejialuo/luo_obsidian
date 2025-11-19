# Language Syntax

## Overview

Beancount is a declarative language. The input consists of a text file containing mainly a
list of **directives**, or **entries**; there is also syntax for defining various
**options**. Each directive begins with an associate *date*, which determines the point in
time at which the directive will apply, and its *type*, which defines which kind of event
this directive represents. All the directives begin with a syntax that looks like this:

```beancount
YYYY-MM-DD <type> ...
```

The format of the date should follow the rule of
[international ISO 8601 standard format](https://www.iso.org/iso-8601-date-and-time-format.html).
For example, `2014-02-03`.

### Accounts

Beancount accumulates commodities in accounts. An account name is a colon-separated list
of capitalized words which begin with a letter, and whose first word must be of
[[basic_concept#Types of Accounts|five account types]].

Each component of the account names begin with a capital letter or a number and are
followed by letters, numbers or dash (`-`) characters. All other characters are
disallowed.

The set of all names of accounts seen in an input file file implicitly define a hierarchy
of accounts (sometimes called a **char-of-accounts**), similarly to how files are
organized in a file system. For example, the following account names:

```txt
Assets:US:BofA:Checking
Assets:US:BofA:Savings
Assets:US:Vanguard:Cash
Assets:US:Vanguard:RGAGX
Assets:Receivables
```

It declares a tree of accounts that looks like this:

```txt
`-- Assets
    |-- Receivables
    `-- US
        |-- BofA
        |   |-- Checking
        |   `-- Savings
        `-- Vanguard
            |-- Cash
            `-- RGAGX
```

### Commodities / Currencies

Accounts contain **currencies**, which we sometimes also call **Commodities**. Like
account names, currency names are recognized by their syntax, though, unlike account
names, they need not be declared before being used. The syntax for a currency is a word
*all* in capital letters.

Technically, a currency name may be up to 24 characters long, and it must start with a
capital letter, must end with with a capital letter or number, and its other characters
must only be capital letters, numbers, or punctuation limited to these characters: `'._-`.

We should use the standard
[ISO 4217 currency code](http://en.wikipedia.org/wiki/ISO_4217#Active_codes) as a
guideline. And there is a [[#Commodity]] directive that can be used to declare currencies.
It is entirely optional.

### Strings

Whenever we need to insert some free text as part of an entry, it should be surrounded by
double-quotes.

### Comments

Any text on a line after the character `;` is ignored. And any line that does not begin as
a valid Beancount syntax directive is silently ignored.

## Directives

### Open

All accounts need to be declared "open" in order to accept amounts posted to them. The
general format of the `Open` directive is:

```beancount
YYYY-MM-DD open Account [ConstraintCurrency,...] ["BookingMethod"]
```

The comma-separated optional list of constraint currencies enforces that all changes
posted to this account are in units of one of the declared currencies. *Specifying a
currency constraint is recommended*.

Another optional declaration for opening accounts is the "booking method", which is the
algorithm that will be invoked if a reducing lot leads to an ambiguous choice of matching
lots from the *inventory*. Currently, the possible values it can take are:

+ **STRICT**: default method
+ **NONE**.

### Close

Similarly to the [[#Open]] directive, there is a `Close` directive that can be used to
tell Beancount that an account has become inactive. The general format of the `Close`
directive is:

```beancount
YYYY-MM-DD close Account
```

Note that a `Close` directive does not currently generate an **implicit zero balance
check**. You may want to add one just before the closing date to ensure that the account
is correctly closed with empty contents.

### Commodity

There is a `Commodity` directive that can be used to declare currencies, financial
instruments, commodities. The general format of the `Commodity` directive is:

```beancount
YYYY-MM-DD commodity Currency
```

The purpose of this directive is to attach commodity-specific metadata fields on it, so
that it can be gathered by plugins later on. For example:

```beancount
1867-07-01 commodity CAD
  name: "Canadian Dollar"
  asset-class: "cash"

2012-01-01 commodity HOOL
  name: "Hooli Corporation Class C Shares"
  asset-class: "stock"
```

### Transactions

Transactions are the most common type of directives. They are slightly different to the
other ones, because they can be followed by a list of postings. Here is an example:

```beancount
2014-05-05 txn "Cafe Mogador" "Lamb tagine with wine"
  Liabilities:CreditCard:CapitalOne         -37.45 USD
  Expenses:Restaurant
```

Instead of using `txn`, we could just use a flag. A flag is used to indicate the status of
a transaction, and the particular meaning of the flag is yours to define. We recommend
using the following interpretation for them:

+ `*`: Completed transaction
+ `!`: Incomplete transaction, needs confirmation or revision.

You can also attach flags to the postings themselves, if you want to flag one of the
transaction’s legs in particular:

```beancount
2014-05-05 * "Transfer from Savings account"
  Assets:MyBank:Checking            -400.00 USD
  ! Assets:MyBank:Savings
```

The general format of a Transaction directive is:

```beancount
YYYY-MM-DD [txn|Flag] [[Payee] Narration] [Flag]
    Account Amount [{Cost}] [@ Price] [Flag]
    Account Amount [{Cost}] [@ Price] ...
```

The lines that follow the first line are for "Postings". You can attach as many postings
as you want to a transaction. For example, a salary entry might look like this:

```beancount
2014-03-19 * "Acme Corp" "Bi-monthly salary payment"
  Assets:MyBank:Checking             3062.68 USD     ; Direct deposit
  Income:AcmeCorp:Salary            -4615.38 USD     ; Gross salary
  Expenses:Taxes:TY2014:Federal       920.53 USD     ; Federal taxes
  Expenses:Taxes:TY2014:SocSec        286.15 USD     ; Social security
  Expenses:Taxes:TY2014:Medicare       66.92 USD     ; Medicare
  Expenses:Taxes:TY2014:StateNY       277.90 USD     ; New York taxes
  Expenses:Taxes:TY2014:SDI             1.20 USD     ; Disability insurance
```

The `Amount` in "Postings" can also be an arithmetic expression using `( ) * / - +`.

#### Payee & Narration

A transaction may have an optional `payee` and/or a `narration`.

The `payee` is a string that represents an external entity that is involved in the
transaction. Payees are sometimes useful on transactions that post amounts to Expense
accounts, whereby the account accumulates a category of expenses from multiple businesses.
A good example is “`Expenses:Restaurant`”, which will include all postings for the various
restaurants one might visit.

A `narration` is a description of the transaction that you write. It can be a comment
about the context, the person who accompanied you, some note about the product you bought.

#### Costs and Prices

If you converted the amount from another currency, you must provide a conversion rate to
balance the transaction. This is done by attaching a `price` to the posting:

```beancount
2012-11-03 * "Transfer to account in Canada"
  Assets:MyBank:Checking            -400.00 USD @ 1.09 CAD
  Assets:FR:SocGen:Checking          436.01 CAD
```

You can also use the `@@` syntax to specify the total cost:

```beancount
2012-11-03 * "Transfer to account in Canada"
  Assets:MyBank:Checking            -400.00 USD @@ 436.01 CAD
  Assets:FR:SocGen:Checking          436.01 CAD
```

However, some commodities that you deposit in accounts must be “held at cost.” This
happens when you want to keep track of the *cost basis* of those commodities. The typical
use case is investing, for example when you deposit shares of a stock in an account.

```beancount
2014-02-11 * "Bought shares of S&P 500"
  Assets:ETrade:IVV                10 IVV {183.07 USD}
  Assets:ETrade:Cash         -1830.70 USD
```

#### Tags

Transactions can be tagged with arbitrary strings:

```beancount
2014-04-23 * "Flight to Berlin" #berlin-trip-2014 #germany
  Expenses:Flights              -1230.27 USD
  Liabilities:CreditCard
```

#### Links

Transactions can also be linked together. You may think of the link as a special kind of
tag that can be used to group together a set of financially related transactions over
time.

```beancount
2014-02-05 * "Invoice for January" ^invoice-pepe-studios-jan14
  Income:Clients:PepeStudios           -8450.00 USD
  Assets:AccountsReceivable

2014-02-20 * "Check deposit - payment from Pepe" ^invoice-pepe-studios-jan14
  Assets:BofA:Checking                  8450.00 USD
  Assets:AccountsReceivable
```

### Balance Assertions

A balance assertion is a way for you to input your statement balance into the flow of
transactions. It tells Beancount to verify that the number of units of a particular
commodity in some account should equal some expected value at some point in time.

If no error is reported, you should have some confidence that the list of transactions
that precedes it in this account is highly likely to be correct. This is useful in
practice.

```beancount
YYYY-MM-DD balance Account Amount
```

It is useful to insert a balance assertion for 0 units just before closing an account,
just to make sure its contents are empty as you close it.

### Notes

A `note` directive is simply used to attach a dated comment to the journal of a particular
account. When you render the journal, the notes should be rendered in context. This can be
useful to record facts and claims associated with a financial event.

```beancount
YYYY-MM-DD note Account Description
```

### Documents

A `document` directive can be used to attach an external file to the journal of an
account. The filename gets rendered as a *browser link* in the journals of the web
interface for the corresponding account.

The general format of the `document` directive is:

```beancount
YYYY-MM-DD document Account PathToDocument
```

### Prices

`price` directives can be used to provide data points for this database. A `price`
directive establishes the rate of exchange between one commodity (the base currency) and
another (the quote currency):

The general format of the `price` directive is:

```beancount
YYYY-MM-DD price Commodity Price
```

## Metadata

You may attach arbitrary data to each of your entries and postings. The syntax for it
looks like this:

```beancount
2013-03-14 open Assets:BTrade:HOOLI
  category: "taxable"
```

Metadata can be attached to any directive type. Keys must begin with a lowercase character
from `a-z` and may contain letters, numbers, dashes and underscores. More, the values can
be any of the following data types:

+ Strings
+ Accounts
+ Currency
+ Dates (`datetime.date`)
+ Tags
+ Numbers (Decimal)
+ Amount (`beancount.core.amount.Amount`)

## Options

There are a few global options that can be set in the input file as well, by using a
special undated `option` directive. The general format of the `option` directive is:

```beancount
option Name Value
```

We could use `bean-doctor list-options` to get all the options. One important option is
`operating_currency`.

By default, Beancount does not treat any of the commodities any different from each other.
In particular, it doesn't know that there's anything special about the most common of
commodities one uses: their currencies.

But useful reports try to reduce all non-currency commodities into one of the main
currencies used.

## Plugins

In order to load  plugin Python modules, use the dedicated `plugin` directive:

```beancount
plugin "beancount.plugins.module_name"
```

The name of a plugin should be the name of a Python module in your `PYTHONPATH`. Plugins
also optionally accept some configuration parameters. These can be provided by an option
final string argument:

```beancount
plugin "beancount.plugins.module_name" "configuration data"
```

## Includes

Include directives are supported. This allows you to split up large input files into
multiple files. The general format is:

```beancount
include Filename
```

The specified path can be an absolute or a relative filename. If the filename is relative,
it is relative to the including filename's directory.
