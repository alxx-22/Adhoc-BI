# Exec Page — screen properties

Paste each formula (after the `=`) into the formula bar for the selected screen.

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
Set(varAnimKey1, If(IsBlank(varAnimKey1), 1, varAnimKey1 + 1));
ClearCollect(
    colManagerSummary,
    ForAll(
        Split(First(PowerBIIntegration.Data).'Manager Feedback Pack', " || "),
        {
            ManagerEmail: Index(Split(Value, " // "), 1).Value,
            TotalOpps:    Value(Index(Split(Value, " // "), 2).Value),
            TrackedOpps:  Value(Index(Split(Value, " // "), 3).Value)
        }
    )
);ClearCollect(
    colUntrackedGBU,
    ForAll(
        Split(First(PowerBIIntegration.Data).'Untracked GBU FLM Summary', " || "),
        {
            Manager:  Index(Split(Value, " // "), 1).Value,
            GBU:      Index(Split(Value, " // "), 2).Value,
            FQ:       Index(Split(Value, " // "), 3).Value,
            Total:    Value(Index(Split(Value, " // "), 4).Value)
        }
    )
);ClearCollect(
    colTrackedGBU,
    ForAll(
        Split(First(PowerBIIntegration.Data).'Tracked GBU FLM Summary', " || "),
        {
            Manager: Index(Split(Value, " // "), 1).Value,
            GBU:     Index(Split(Value, " // "), 2).Value,
            FQ:      Index(Split(Value, " // "), 3).Value,
            Total:   Value(Index(Split(Value, " // "), 4).Value)
        }
    )
);
ClearCollect(
    colWonGBU,
    ForAll(
        Split(First(PowerBIIntegration.Data).'Won Sum by Manager GBU', " || "),
        {
            Manager: Index(Split(Value, " // "), 1).Value,
            GBU:     Index(Split(Value, " // "), 2).Value,
            Total:   Value(Index(Split(Value, " // "), 3).Value)
        }
    )
);

ClearCollect(
    colWonGBUSummary,
    ForAll(
        GroupBy(colWonGBU, 'GBU', 'GBURows'),
        {GBU: GBU, Total: Sum(GBURows, Total)}
    )
);

Set(varWonTotal, Sum(colWonGBU, Total));

Set(
    varWonCycleText,
    Concat(
        Sequence(CountRows(colWonGBUSummary)),
        With(
            {
                i:    Value,
                n:    CountRows(colWonGBUSummary),
                row:  Index(colWonGBUSummary, Value),
                durS: Text(CountRows(colWonGBUSummary) * 2.5, "[$-en-US]0.0")
            },
            With(
                {
                    sf: (i - 1) / n,
                    ef: i / n,
                    gbuLabel: Substitute(Substitute(Substitute(Substitute(
                        Index(colWonGBUSummary, Value).GBU,
                        "&", "&amp;"), "'", " "), "<", "&lt;"), ">", "&gt;"),
                    gbuVal: If(row.Total >= 1000000,
                        "$" & Text(row.Total / 1000000, "[$-en-US]0.0") & "M",
                        If(row.Total >= 1000,
                            "$" & Text(row.Total / 1000, "[$-en-US]0") & "K",
                            "$" & Text(row.Total, "[$-en-US]0")))
                },
                "<text x='16' y='118' font-family='Arial, Helvetica, sans-serif' font-size='10' font-weight='400' fill='#606a70' opacity='" & If(i = 1, "1", "0") & "'>" &
                gbuLabel & ": <tspan fill='#7764FC' font-weight='700'>" & gbuVal & "</tspan>" &
                If(i = 1,
                    "<animate attributeName='opacity' values='1;1;0;0' keyTimes='0;" & Text(ef - 0.02, "[$-en-US]0.000") & ";" & Text(ef, "[$-en-US]0.000") & ";1' dur='" & durS & "s' begin='1.5s' repeatCount='indefinite'/>",
                    If(i = n,
                        "<animate attributeName='opacity' values='0;0;1;1' keyTimes='0;" & Text(sf - 0.02, "[$-en-US]0.000") & ";" & Text(sf, "[$-en-US]0.000") & ";1' dur='" & durS & "s' begin='1.5s' repeatCount='indefinite'/>",
                        "<animate attributeName='opacity' values='0;0;1;1;0;0' keyTimes='0;" & Text(sf - 0.02, "[$-en-US]0.000") & ";" & Text(sf, "[$-en-US]0.000") & ";" & Text(ef - 0.02, "[$-en-US]0.000") & ";" & Text(ef, "[$-en-US]0.000") & ";1' dur='" & durS & "s' begin='1.5s' repeatCount='indefinite'/>"
                    )
                ) &
                "</text>"
            )
        )
    )
);
ClearCollect(
    colPivotData,
    ForAll(
        colWonGBU,
        With(
            {
                mgr: Manager,
                gbu: GBU,
                wonAmt: Total
            },
            {
                Manager:      mgr,
                GBU:          gbu,
                WonVal:       wonAmt,
                TrackedVal:   IfError(Sum(Filter(colTrackedGBU,   Manager = mgr && GBU = gbu), Total), 0),
                UntrackedVal: IfError(Sum(Filter(colUntrackedGBU, Manager = mgr && GBU = gbu), Total), 0)
            }
        )
    )
);
Set(varMaxUntracked, Max(colPivotData, UntrackedVal));
Clear(colPivotTable);
ForAll(
    SortByColumns(Distinct(colPivotData, Manager), "Value", SortOrder.Ascending),
    Collect(
        colPivotTable,
        {
            RowType:      "header",
            Manager:      Value,
            GBU:          "",
            WonVal:       Sum(Filter(colPivotData, Manager = Value), WonVal),
            TrackedVal:   Sum(Filter(colPivotData, Manager = Value), TrackedVal),
            UntrackedVal: Sum(Filter(colPivotData, Manager = Value), UntrackedVal),
            SortKey:      Value & "_0"
        }
    );
    Collect(
        colPivotTable,
        ForAll(
            SortByColumns(Filter(colPivotData, Manager = Value), "UntrackedVal", SortOrder.Descending),
            {
                RowType:      "data",
                Manager:      Manager,
                GBU:          GBU,
                WonVal:       WonVal,
                TrackedVal:   TrackedVal,
                UntrackedVal: UntrackedVal,
                SortKey:      Manager & "_1_" & Text(1000000000 - UntrackedVal, "[$-en-US]0000000000")
            }
        )
    )
);
ClearCollect(
    colAAOppIds,
    'Attach Attack'
);

ClearCollect(
    colRepCompletion,
    ForAll(
        GroupBy(
            PowerBIIntegration.Data,
            'Feedback Progress',
            'OppRows'
        ),
        {
            Label: 'Feedback Progress',
            TotalOpps: CountRows(OppRows),
            Completed: CountRows(
                Filter(
                    OppRows,
                    'HPE Opportunity Id' in colAAOppIds.'Opp ID'
                )
            ),
            PctComplete: If(
                CountRows(OppRows) > 0,
                Round(
                    CountRows(Filter(
                        OppRows,
                        'HPE Opportunity Id' in colAAOppIds.'Opp ID'
                    )) / CountRows(OppRows) * 100,
                    0
                ),
                0
            )
        }
    )
);
ClearCollect(colHasRecord,
    Filter('Attach Attack', !IsBlank('Opp ID'))
);
```
