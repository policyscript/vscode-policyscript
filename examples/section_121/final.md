# § 121 - Exclusion of gain from sale of principal residence

**Definitions**

```law
Person:
  id: integer

PropertyOwnedPeriod:
  owner:      Person
  occupant:   Person
  start_date: date
  end_date:   date

PropertySaleOrExchange:
  ownages:  PropertyOwnedPeriod list
  gain:     money
  date:     date
```

**Inputs**

```law
taxpayer:         Person
sale_or_exchange: PropertySaleOrExchange
```

**Outputs**

```law
can_exclude_gain_from_gross:    condition
total_gain_excluded_from_gross: money
```

- ## (a) Exclusion

  Gross income shall not include gain from the sale or exchange of property if,
  during the 5-year period ending on the date of the sale or exchange, such
  property has been owned and used by the taxpayer as the taxpayer’s principal
  residence for periods aggregating 2 years or more.

  **Locals**

  ```law
  time_as_principal_residence:     integer
  5_years_before_sale_or_exchange: date
  start_date_or_5_years_before:    date
  ```

  **Code**

  ```law
  # This is the sum of time that the taxpayer used the property as their
  # principal residence.
  set time_as_principal_residence to 0

  # Set a date that is 5 years prior to the sale or exchange date.
  set 5_years_before_sale_or_exchange to sale_or_exchange.date - 5 years

  # For each ownage in the list of ownages:
  for ownage in sale_or_exchange.ownages {

    # Only add to time as principal resident if the ownage owner matches the
    # taxpayer, and if the end date is within 5 years (therefore the date range
    # falls within the last 5 years).
    if ownage.owner = taxpayer and ownage.user = taxpayer and
      ownage.end_date >= 5_years_before_sale_or_exchange {

      # If start date of ownage is prior to 5 years before sale or exchange,
      # then start counting at 5 years instead of start date.
      if ownage.start_date < 5_years_before_sale_or_exchange {
        set start_date_or_5_years_before to 5_years_before_sale_or_exchange
      }

      # Otherwise, start counting at the actual start date.
      else {
        set start_date_or_5_years_before to ownage.start_date
      }

      # Add the difference between start and end to the principal residence sum.
      set time_as_principal_residence to time_as_principal_residence +
        (ownage.end_date - start_date_or_5_years_before)
    }
  }

  # If time spent as principal resident is greater than 2 years, can exclude
  # gain from gross.
  if time_as_principal_residence >= 2 years {
    set can_exclude_gain_from_gross to true
  }
  else {
    set can_exclude_gain_from_gross to false
  }
  ```

- ## (b) Limitations

  - ### (1) In general

    The amount of gain excluded from gross income under subsection (a) with respect
    to any sale or exchange shall not exceed $250,000.

    **Locals**

    ```law
    maximum_gain_exclusion: money
    ```

    **Code**

    ```law
    set maximum_gain_exclusion to $250_000.00

    # Initially, no gain can be excluded from gross.
    set total_gain_excluded_from_gross to $0.00

    # If we are able to exclude gain from gross:
    if can_exclude_gain_from_gross {

      # If gain is greater than maximum gain exclusion, cap it.
      if sale_or_exchange.gain > maximum_gain_exclusion {
        set total_gain_excluded_from_gross to maximum_gain_exclusion
      }
      else {
        set total_gain_excluded_from_gross to sale_or_exchange.gain
      }
    }
    ```
