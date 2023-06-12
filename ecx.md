The very first step of the project was to collect the historical data from ECX website. The extracted data in csv format contained the following headers.

> Trade Date, Symbol, Warehouse, Production Year, Opening Price, Closing
> Price, High, Low, Change, Percetntage Change, Volume (Ton)

The following task was to clean and load the data for further exploratory analysis. However this presented a challenge in its own as the symbol (e.g. LWWAR1) contained multiple attributes that should be distributed to be in individual columns of their own. Therefore a rule had to be established as to how to extract individual attributes from the symbol column.

The data normalization was done using Power Query and its query language M. The objective here was to extract the following attributes.

> Grade [1-10 with or without undergrade(UG) tag]<br>
> Market [Local and Export] <br>
> Distinction [Speciality, Commercial or Domestic]<br>
> Processing [Washed, Unwashed and Semi-washed]<br>
> Origin - Harvest areas<br>

<details><summary>Find the code here</summary><code>
    
            let
        Source = HistoricalPrices,
        #"Removed Other Columns" = Table.SelectColumns(Source,{"Index", "Symbol"}),
        #"Inserted Text Length" = Table.AddColumn(#"Removed Other Columns", "Length", each Text.Length([Symbol]), Int64.Type),
        #"Inserted First Characters" = Table.AddColumn(#"Inserted Text Length", "First Characters", each Text.Start([Symbol], 1), type text),
        #"Added Conditional Column" = Table.AddColumn(#"Inserted First Characters", 
        "Market", each if [First Characters] = "L" then "Local" else if [First Characters] = "2" then null else "Export"),
        #"Inserted Text Range" = Table.AddColumn(#"Added Conditional Column", "Text Range", each Text.Middle([Symbol], 1, 1), type text),
        #"Renamed Columns" = Table.RenameColumns(#"Inserted Text Range",{{"Text Range", "SecondChar"}, {"First Characters", "FirstChar"}}),
        #"Added Custom" = Table.AddColumn(#"Renamed Columns", "Processing", each if [FirstChar] = "W" then "Washed"
    else if [FirstChar] = "U" then "Unwashed"
    else if [FirstChar] = "S" then "Semi Washed"
    else if [FirstChar] = "L" and [SecondChar]="U" then "Unwashed"
    else if [FirstChar] = "L" and [SecondChar] = "W" then "Washed"
    else "NA"),
        #"Duplicated Column" = Table.DuplicateColumn(#"Added Custom", "Symbol", "Symbol - Copy"),
        #"Split Column by Character Transition" = Table.SplitColumn
        (#"Duplicated Column", "Symbol - Copy", Splitter.SplitTextByCharacterTransition((c) => 
        not List.Contains({"0".."9"}, c), {"0".."9"}), {"Symbol - Copy.1", "Symbol - Copy.2"}),
        #"Renamed Columns1" = Table.RenameColumns(#"Split Column by Character Transition",{{"Symbol - Copy.1", "SC1"}, {"Symbol - Copy.2", "Grade1"}}),
        #"Replaced Value" = Table.ReplaceValue(#"Renamed Columns1",null,"UG",Replacer.ReplaceValue,{"Grade1"}),
        #"Duplicated Column1" = Table.DuplicateColumn(#"Replaced Value", "SC1", "SC1 - Copy"),
        #"Removed Columns" = Table.RemoveColumns(#"Duplicated Column1",{"SC1 - Copy"}),
        #"Inserted Last Characters" = Table.AddColumn(#"Removed Columns", "Last Characters", each Text.End([SC1], 2), type text),
    
    
        #"Reigon Extraction" = Table.AddColumn(#"Inserted Last Characters", "ReigonCode", each if Text.Contains([Symbol],"yc",Comparer.OrdinalIgnoreCase) then "YC"
    else if Text.Contains([Symbol],"kt",Comparer.OrdinalIgnoreCase) then "kt"
    else if Text.Contains([Symbol],"am",Comparer.OrdinalIgnoreCase) then "am"
    else if Text.Contains([Symbol],"wt",Comparer.OrdinalIgnoreCase) then "wt"
    else if Text.Contains([Symbol],"ge",Comparer.OrdinalIgnoreCase) then "ge"
    else if Text.Contains([Symbol],"war",Comparer.OrdinalIgnoreCase) then "war"
    else if Text.Contains([Symbol],"ib",Comparer.OrdinalIgnoreCase) then "ib"
    else if Text.Contains([Symbol],"bl",Comparer.OrdinalIgnoreCase) then "bl"
    else if Text.Contains([Symbol],"gd",Comparer.OrdinalIgnoreCase) then "gd"
    else if Text.Contains([Symbol],"yk",Comparer.OrdinalIgnoreCase) then "yk"
    else if Text.Contains([Symbol],"eg",Comparer.OrdinalIgnoreCase) then "eg"
    else if Text.Contains([Symbol],"wg",Comparer.OrdinalIgnoreCase) then "wg"
    else if Text.Contains([Symbol],"zg",Comparer.OrdinalIgnoreCase) then "zg"
    else if Text.Contains([Symbol],"awi",Comparer.OrdinalIgnoreCase) then "awi"
    else if Text.Contains([Symbol],"wl",Comparer.OrdinalIgnoreCase) then "wl"
    else if Text.Contains([Symbol],"sh",Comparer.OrdinalIgnoreCase) then "sh"
    else if Text.Contains([Symbol],"gm",Comparer.OrdinalIgnoreCase) then "gm"
    else if Text.Contains([Symbol],"bb",Comparer.OrdinalIgnoreCase) then "bb"
    else if Text.Contains([Symbol],"wn",Comparer.OrdinalIgnoreCase) then "wn"
    else if Text.Contains([Symbol],"kc",Comparer.OrdinalIgnoreCase) then "kc"
    else if Text.Contains([Symbol],"ga",Comparer.OrdinalIgnoreCase) then "ga"
    else if Text.Contains([Symbol],"sd",Comparer.OrdinalIgnoreCase) then "sd"
    else if Text.Contains([Symbol],"lm",Comparer.OrdinalIgnoreCase) then "lm"
    else if Text.Contains([Symbol],"kf",Comparer.OrdinalIgnoreCase) then "kf"
    else if Text.Contains([Symbol],"jm",Comparer.OrdinalIgnoreCase) then "jm"
    else if Text.Contains([Symbol],"yk",Comparer.OrdinalIgnoreCase) then "yk"
    else if Text.Contains([Symbol],"an",Comparer.OrdinalIgnoreCase) then "an"
    else if Text.Contains([Symbol],"bm",Comparer.OrdinalIgnoreCase) then "bm"
    else if Text.Contains([Symbol],"kw",Comparer.OrdinalIgnoreCase) then "kw"
    else if Text.Contains([Symbol],"ew",Comparer.OrdinalIgnoreCase) then "ew"
    else if Text.Contains([Symbol],"tp",Comparer.OrdinalIgnoreCase) then "tp"
    else if Text.Contains([Symbol],"lk",Comparer.OrdinalIgnoreCase) then "lk"
    else if Text.Contains([Symbol],"hr",Comparer.OrdinalIgnoreCase) then "hr"
    else if Text.Contains([Symbol],"fr",Comparer.OrdinalIgnoreCase) then "fr"
    else if Text.Contains([Symbol],"aa",Comparer.OrdinalIgnoreCase) then "aa"
    else if Text.Contains([Symbol],"dd",Comparer.OrdinalIgnoreCase) then "dd"
    else if Text.Contains([Symbol],"gj",Comparer.OrdinalIgnoreCase) then "gj"
    else if Text.Contains([Symbol],"bpaa",Comparer.OrdinalIgnoreCase) then "bpaa"
    else if Text.Contains([Symbol],"bpdd",Comparer.OrdinalIgnoreCase) then "bpdd"
    else if Text.Contains([Symbol],"bp",Comparer.OrdinalIgnoreCase) then "bp"
    else if Text.Contains([Symbol],"sk",Comparer.OrdinalIgnoreCase) then "sk"
    else if Text.Contains([Symbol],"dl",Comparer.OrdinalIgnoreCase) then "dl"
    else null),
        #"Replaced Value1" = Table.ReplaceValue(#"Reigon Extraction","BB","--",Replacer.ReplaceText,{"SC1"})
    in
        #"Replaced Value1"

  </code></details>


