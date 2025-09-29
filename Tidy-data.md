Tidy data
================
2025-09-27

This document will show how to tidy data

## Pivot longer

``` r
pulse_df = 
  read_sas("data_import_examples/public_pulse_data.sas7bdat") |> 
  janitor::clean_names() |> 
  pivot_longer(
    cols = bdi_score_bl:bdi_score_12m,
    names_to = "visit",
    values_to = "bdi_score",
    names_prefix = "bdi_score_"
  ) |> 
  mutate(
    visit = replace(visit, visit == "bl", "00m") 
  ) |> 
  relocate(id, visit)
```

Do one more example

``` r
litters_df = 
  read_csv("data_import_examples/FAS_litters.csv", na = c("NA", ".","")) |> 
  janitor::clean_names() |> 
  pivot_longer(
    cols = gd0_weight:gd18_weight,
    names_to = "gd_time",
    values_to = "weight",
  ) |> 
  mutate(
    gd_time = case_match(
      gd_time,
      "gd0_weight" ~ 0 ,
      "gd18_weight" ~ 18
    )
  )
```

    ## Rows: 49 Columns: 8
    ## ── Column specification ────────────────────────────────────────────────────────
    ## Delimiter: ","
    ## chr (2): Group, Litter Number
    ## dbl (6): GD0 weight, GD18 weight, GD of Birth, Pups born alive, Pups dead @ ...
    ## 
    ## ℹ Use `spec()` to retrieve the full column specification for this data.
    ## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

## pivot wider

``` r
analysis_df = 
  tibble(group = c("treatment","treatment","control","control"),
         time = c("pre","post","pre","post"),
         mean = c(4, 10, 4.2, 5)
         )
```

Pivot wider for human readability.

``` r
analysis_df |> 
  pivot_wider(
    names_from = time,
    values_from = mean
  ) |> 
  knitr::kable()
```

| group     | pre | post |
|:----------|----:|-----:|
| treatment | 4.0 |   10 |
| control   | 4.2 |    5 |

## Bind tables

``` r
fellowship_ring = 
  read_excel("data_import_examples/LotR_Words.xlsx", range = "B3:D6") |> 
  mutate(movie = "fellowship_ring")

two_towers = 
  read_excel("data_import_examples/LotR_Words.xlsx", range = "F3:H6") |> 
  mutate(movie = "two_towers")

return_of_the_king = 
  read_excel("data_import_examples/LotR_Words.xlsx", range = "J3:L6") |> 
  mutate(movie = "return_of_the_king")

lotr_df = bind_rows(fellowship_ring, two_towers, return_of_the_king) |> 
  pivot_longer(
    cols = Female:Male,
    names_to = "sex",
    values_to = "number_of_words_spoken"
  ) |> 
  relocate(movie) |> 
  mutate(Race = str_to_lower(Race))
```

## join FAS datasets

Import `litters` dataset:

``` r
litters_df = read_csv("data_import_examples/FAS_litters.csv", na = c("NA", ".", "")) |> 
  janitor::clean_names() |> 
  mutate(wt_gain = gd18_weight - gd0_weight) |> 
  separate(
    group, into = c("dose", "day_of_treatment"), sep = 3
  )
```

    ## Rows: 49 Columns: 8
    ## ── Column specification ────────────────────────────────────────────────────────
    ## Delimiter: ","
    ## chr (2): Group, Litter Number
    ## dbl (6): GD0 weight, GD18 weight, GD of Birth, Pups born alive, Pups dead @ ...
    ## 
    ## ℹ Use `spec()` to retrieve the full column specification for this data.
    ## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

Import `pups` dataset:

``` r
pups_df = read_csv("data_import_examples/FAS_pups.csv", skip = 3, na = c("NA", ".", "")) |> 
  janitor::clean_names() |> 
  mutate(
    sex = case_match(
    sex, 
    1 ~ "male",
    2 ~ "female"
  )
  )
```

    ## Rows: 313 Columns: 6
    ## ── Column specification ────────────────────────────────────────────────────────
    ## Delimiter: ","
    ## chr (1): Litter Number
    ## dbl (5): Sex, PD ears, PD eyes, PD pivot, PD walk
    ## 
    ## ℹ Use `spec()` to retrieve the full column specification for this data.
    ## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

join the datasets!

``` r
fas_df = 
  left_join(pups_df, litters_df, by = "litter_number") |> 
  relocate(litter_number, dose, day_of_treatment)
```
