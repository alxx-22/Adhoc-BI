# Summer Score - screen properties

Select the screen in the tree view, pick each property in the dropdown,
and paste the formula (everything after the `=`) into the formula bar.

## Fill

```
RGBA(247, 247, 247, 1)
```

## LoadingSpinnerColor

```
RGBA(1, 169, 130, 1)
```

## OnVisible

```
Set(varAnimKey1, If(IsBlank(varAnimKey1), 1, varAnimKey1 + 1));ClearCollect(
    colEngagementMetrics,
    ForAll(
        Split(First(PowerBIIntegration.Data).PackedEngagement, "||"),
        {
            UniqueUser:                     Index(Split(Value, "//"), 1).Value,
            WeeklyScore:                    Index(Split(Value, "//"), 2).Value,
            IBNSPoints:                     Index(Split(Value, "//"), 3).Value,
            BookingRate:                    Index(Split(Value, "//"), 4).Value,
            AttachUpsellPoints:             Index(Split(Value, "//"), 5).Value,
            BacklogPoints:                  Index(Split(Value, "//"), 6).Value,
            Manager:                        Index(Split(Value, "//"), 7).Value,
            LeadingEdgeCompletionPoints:    Index(Split(Value, "//"), 8).Value,
            DisplayName:                    Index(Split(Value, "//"), 9).Value,
            Country:                        Index(Split(Value, "//"), 10).Value
        }
    )
);

ClearCollect(
    colSorted,
    SortByColumns(
        AddColumns(colEngagementMetrics,
            'TotalPts',
            Value(IBNSPoints) + Value(AttachUpsellPoints) + Value(BacklogPoints) + Value(LeadingEdgeCompletionPoints)
        ),
        "TotalPts",    SortOrder.Descending,
        "WeeklyScore", SortOrder.Descending
    )
);

ClearCollect(
    colEngagementRanked,
    ForAll(
        Sequence(CountRows(colSorted)),
        {
            Rank:                        Value,
            UniqueUser:                  Index(colSorted, Value).UniqueUser,
            WeeklyScore:                 Index(colSorted, Value).WeeklyScore,
            IBNSPoints:                  Index(colSorted, Value).IBNSPoints,
            BookingRate:                 Index(colSorted, Value).BookingRate,
            AttachUpsellPoints:          Index(colSorted, Value).AttachUpsellPoints,
            BacklogPoints:               Index(colSorted, Value).BacklogPoints,
            Manager:                     Index(colSorted, Value).Manager,
            LeadingEdgeCompletionPoints: Index(colSorted, Value).LeadingEdgeCompletionPoints,
            DisplayName:                 Index(colSorted, Value).DisplayName,
            Country:                     Index(colSorted, Value).Country
        }
    )
);
```
