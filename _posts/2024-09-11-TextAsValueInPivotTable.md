# Text as value in MS Excel Pivot table 

I was doing an analysis of who is assigned to which all client projects in my team based on timesheet data.

Instead of doing it manually I thought there has to be a way to do this in Excel.

I searched on Google and found [this link](https://www.mrexcel.com/excel-tips/pivot-table-with-text-in-values-area/) which satisfied my requirement.

I used =CONCATENATEX(Values(Table1[Code]), Table1[Code], ", ") to find out a unique concatenated list of projects assigned to each member in the Pivot.

Learnt a good nifty trick for MS Excel Pivot table.