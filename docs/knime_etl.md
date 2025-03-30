# Extract, Transform, Load (ETL) using KNIME and SQL Server

## Summary

![](images/project_1_diagram.png){width="1000"}

!!! abstract ""
    :fontawesome-solid-triangle-exclamation: **Problem:** The analytics team spent a lot of time cleaning the sales and inventory data that they received in Excel files every week. 

    :material-lightbulb-on-10: **Solution:** I used KNIME to develop an ETL program to extract the data from the Excel files, transform the data, then load the transformed data into SQL Server.

    :octicons-graph-16: **Results:** The analytics team can simply query data from SQL Server instead of wrangling data from thousands of Excel files. 
    

## KNIME
KNIME is a low-code data processing app where you can chain “nodes” to create programs. A node is like a function in a programming language. 

![](images/knime_screenshot.png){width="600"}
/// caption
Screenshot of KNIME.
///

## Definitions
![](images/definitions.png){width="1000"}

`f`
:   An Excel file containing sales and inventory data for one week. 

`f.data`
:   The data table extracted from `f`. All columns in `f.data` are the string data type. 

`T`
:   A sequence of data transformations. These include:  

    - Transforming column contents using regular expressions.  
    - Converting column data types.  

`T(f.data)`
:   The resulting table after applying `T` to `f.data`. 

`all_data`
:   A database table that stores `T(f.data)` for every Excel file `f`.

**match**
:   A table `A` matches a table `B` if `A` and `B` have the same column names and data types.
    `T(f.data)` can only be loaded into `all_data` if `T(f.data)` matches `all_data`.

`L_all`
:   A list of all the Excel file names.

`L_loaded`
:   A list of the Excel file names `f.name` such that `T(f.data)` has been loaded into `all_data`. 

## Pseudocode of the ETL program
<style>
.code {
    background-color: #f5f5f5; 
}
</style>

<pre>
<b>Algorithm</b> ETL_program(<span class="code">L</span>):  
    <b>Input:</b>   
        <b><span class="code">L</span>:</b> A list of Excel file names.
    <b>Output:</b> Nothing. 

	<b>for</b> each Excel file name <span class="code">f.name</span> in <span class="code">L</span>:
	    <span class="code">f.data</span> = Extract the data table from <span class="code">f</span>.  
	    <span class="code">T(f.data)</span> = Apply each transformation in <span class="code">T</span> to <span class="code">f.data</span>. 
        <b>try:</b>
	        Compare the column names and data types in <span class="code">T(f.data)</span> and 
            <span class="code">all_data</span>. If <span class="code">T(f.data)</span> doesn’t match <span class="code">all_data</span>, raise an 
            exception describing the mismatching columns.
	    <b>except</b> Exception as <span class="code">error</span>:
	        Print <span class="code">f"{f.name}: {error}"</span> (a formatted string).   
	    <b>else:</b>
            <b>if</b> <span class="code">f.name</span> is not in <span class="code">L_loaded</span>: 
	            Load <span class="code">T(f.data)</span> into <span class="code">all_data</span>.
	            Append <span class="code">f.name</span> to <span class="code">L_loaded</span>.   
</pre>
<br>
`(L_all − L_loaded)` is the list of the Excel file names `f.name` such that `T(f.data)` hasn't been 
loaded into `all_data`. 

After executing `ETL_program(L_all - L_loaded)`, I execute the following steps manually:
<pre>
<b>while</b> <span class="code">(L_all - L_loaded)</span> is not empty:
    <span class="code">f.name = (L_all - L_loaded)[0]</span>.
        Execute <span class="code">ETL_program([f.name])</span>. 
        Use the printed error message to update the data 
        transformations in <span class="code">T</span>.
    Execute <span class="code">ETL_program(L_all - L_loaded)</span>. 
</pre>


## Example execution of the ETL program

`T`:
:   1. Convert `date` to the date data type.    
    2. Convert `column_a` to the integer data type.   

**1. Execute `ETL_program([f_1.name])`:**  

![](images/etl_program_1.png){width="800"}
/// caption
`T(f_1.data)` can't be loaded into `all_data` because `"missing"` in `column_a` can't be converted to an integer. 
///

**2. Update `T`:**  

`T`: 
:   1. Convert `date` to the date data type.    
    2. <span style="background-color:#d8f5e6">In `column_a`, replace all cells matching the regular expression `“^missing$”` with `null`.</span>
    3. Convert `column_a` to the integer data type.

**3. Execute `ETL_program([f_1.name])`:**    

![](images/etl_program_2.png){width="1000"}
/// caption
After updating `T`, `T(f_1.data)` is loaded into `all_data`.
///
