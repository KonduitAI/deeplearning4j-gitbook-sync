# Visualization

## Utilities

### HtmlAnalysis

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/datavec/datavec-api/src/main/java/org/datavec/api/transform/ui/HtmlAnalysis.java)

**createHtmlAnalysisString**

```text
public static String createHtmlAnalysisString(DataAnalysis analysis) throws Exception 
```

Render a data analysis object as a HTML file. This will produce a summary table, along charts for numerical columns. The contents of the HTML file are returned as a String, which should be written to a .html file.

* param analysis Data analysis object to render
* see \#createHtmlAnalysisFile\(DataAnalysis, File\)

**createHtmlAnalysisFile**

```text
public static void createHtmlAnalysisFile(DataAnalysis dataAnalysis, File output) throws Exception 
```

Render a data analysis object as a HTML file. This will produce a summary table, along charts for numerical columns

* param dataAnalysis Data analysis object to render
* param output Output file \(should have extension .html\)

### HtmlSequencePlotting

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/datavec/datavec-api/src/main/java/org/datavec/api/transform/ui/HtmlSequencePlotting.java)

A simple utility for plotting DataVec sequence data to HTML files. Each file contains only one sequence. Each column is plotted separately; only numerical and categorical columns are plotted.

**createHtmlSequencePlots**

```text
public static String createHtmlSequencePlots(String title, Schema schema, List<List<Writable>> sequence)
                    throws Exception 
```

Create a HTML file with plots for the given sequence.

* param title Title of the page
* param schema Schema for the data
* param sequence Sequence to plot
* return HTML file as a string

**createHtmlSequencePlotFile**

```text
public static void createHtmlSequencePlotFile(String title, Schema schema, List<List<Writable>> sequence,
                    File output) throws Exception 
```

Create a HTML file with plots for the given sequence and write it to a file.

* param title Title of the page
* param schema Schema for the data
* param sequence Sequence to plot

