# Covid Saarani (कोविड सारणी)

This is the documentation for Covid Saarani (कोविड सारणी), which is an API /
Dashboard for India's COVID-19 data fetched from Union Government's sources.
Covid Saarani's data and workflow is _entirely_ on GitHub.

Prefer using this over directly spamming and parsing Government websites.

- Ease of use: Data from different streams is clubbed into one single structure.

- Data is the same as shown on websites like that of MoHFW.

- Advantage of the reliability of GitHub servers over government portals.

- Avoid getting IP blocked by government servers, and also reduce load on them.

## Dashboard

https://bit.ly/covid-saarani

## API fetch

Currently, there is no REST API implemented. For the time being, we can take
advantage of the symlink in the [सारणी](https://github.com/covid-saarani/saarani)
repo. For example, in Python we can do the following:

```python
import requests

url = "https://raw.githubusercontent.com/covid-saarani/saarani/main/"
data = requests.get(url + requests.get(url + "latest.json").text).json()
```

Since [लिपिक](https://github.com/covid-saarani/lipik) works on an hourly basis,
one doesn't need to fetch repeatedly. Union Government's data anyways doesn't
change much throughout the day.

## Contents of the API

The JSON returned by API contains:

- Active, recovered, deaths, and confirmed/total cases for states, containing
    - Current day total,
    - Previous day total,
    - Delta / new cases.
    - Percent ratio of active/recovered/death cases compared to confirmed.

- Vaccination data for states, containing
    - Number of vaccination centers.
    - Age wise breakup of doses (All, 18+, and 15-18 age groups).
    - Dose wise breakup (1st dose, 2nd dose, 3rd dose, total doses).
    - Total and new doses for each type of dose.

- Hindi name for each state to aid in localisation.

- Helplines and state fund donation links.

- District weekly data for each state, containing:
    - Number of vaccination centers,
    - Positivity rate,
    - Percent ratio of RTPCR tests compared to overall tests / samples taken,
    - Percent ratio of Rapid Antigen Tests compared to overall tests taken.

## Structure / Documentation of the API

### Overall collapsed look:

```yaml
    "timestamp": (dict)
    "Full state name": (dict) (National stats -> "All")
```

### Detailed look:

```yaml
"timestamp": {
    "cases": {
        "date": (str) Date corresponding to the case count.
        "as_on": (str) Data collection timestamp (data is as on this date and time).
        "last_updated_unix": (int) Epoch time of the last update time of data by the Government.
        "last_fetched_unix": (int) Epoch time of the last fetch time of data by Covid Saarani.
    }

    "vaccination": {
        "date": (str) Date corresponding to the vaccination count.
        "as_on": (str) Data collection timestamp (data is as on this date and time).
        "last_fetched_unix": (int) Epoch time of the last fetch time of data by Covid Saarani.
    }

    "districts": {
        "week": (str) Week of data (Last 7 days generally).
        "last_fetched_unix": (int) Epoch time of the last fetch time of data by Covid Saarani.
    }
}
```

```yaml
"Full State Name": {
    "abbr": (str) Two letter abbreviation for the state.
    "hindi": (str) Hindi/Devanagari name for the state.
    "helpline": (str) Government helplines for COVID.
    "donate": (str) Link to state funds for tackling COVID.

    "confirmed": {
        "current": (int) Total case count as of now.
        "previous": (int) Total case count on the previous day.
        "new": (int) New confirmed cases in the last 24 hours.
    }

    "active": {
        "current": (int) Active case count as of now.
        "previous": (int) Active case count on the previous day.
        "new": (int) New active cases in the last 24 hours.
        "ratio_pc": (float, 5 digit precision) % of active cases in the total cases.
    }

    "recovered": {
        "current": (int) recovered case count as of now.
        "previous": (int) recovered case count on the previous day.
        "new": (int) New recovered cases in the last 24 hours.
        "ratio_pc": (float, 5 digit precision) % of recovered cases in the total cases.
    }

    "deaths": {
        "current": (int) Death count as of now.
        "previous": (int) Death count on the previous day.
        "new": (int) New deaths in the last 24 hours.
        "reconciled": (int) Reconciled deaths which were added to current day count.
        "ratio_pc": (float, 5 digit precision) % of deaths in the total cases.
    }

    "vaccination": (dict) Described below.
    "districts": (dict) Described below.
}
```

Note: For national stats, `All` is used in the state name field.
Also, the district dict will be empty.

There is also a `Miscellaneous` key containing miscellaneous data.

---

```yaml
"vaccination": {
    "centers": (int) Total number of vaccination centers in the state.
    "all_ages": (dict) Containing vaccination dose data for all ages. Dict described below.
    "18+": (dict) Containing vaccination dose data for adults.
    "15-18": (dict) Containing vaccination dose data for children (15-18).
}
```

```yaml
"all_ages/18+/15-18": {
    "all_doses": {
        "total": (int) Combined sum (1st+2nd+3rd) of total doses administered.
        "new": (int) Combined sum (1st+2nd+3rd) of new doses administered today.
    }

    "1st_dose": {
        "total": (int) Total number of first doses administered.
        "new": (int) New first doses administered today.
    }

    "2nd_dose": {
        "total": (int) Total number of second doses administered.
        "new": (int) New second doses administered today.
    }

    "3rd_dose": {
        "total": (int) Total number of precaution doses administered.
        "new": (int) New precaution doses administered today.
    }
}
```

---

```yaml
"districts": {
    "Full district name": {
        "centers": (int) Total number of vaccination centers in the district.
        "rat_pc": (int or float) % of RAT tests out of total tests done.
        "rtpcr_pc": (int or float) % of RTPCR tests out of total tests done.
        "positivity_rate": (float) The positivity rate of the district for the past week (See timestamp).
    }
}
```

For national stats, there will be a district named `Aggregate`, containing the
aggregate stats for all the districts. Thus, the national positivity rate and
other stats can be retrieved.

---
