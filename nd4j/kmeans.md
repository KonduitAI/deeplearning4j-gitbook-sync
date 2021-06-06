# K-Means Clustering with ND4J

In this tutorial we will show you how to use Nd4j's NDArray to re-create the classic k-means algorithm. 

## K-Means Basics
K-means clustering is an unsupervised algorithm that groups data into `k` clusters by minimizing intra-cluster variance. The choice of k is left up to the user, and while there are several heuristics for finding the appropriate `k` value, we will not discuss that in this tutorial. 

**Resource**: https://en.wikipedia.org/wiki/K-means_clustering 

## The Algorithm: 

K-means is a simple algorithm in three steps. An initialization step is followed by two steps that occur in loop until convergence:

1. Init: Choose k initial centroids.
2. Assign: Assign each datapoint to its closest centroid.
3. Update: Recalculate the centroids of each cluster based on the mean of newly assigned data.
4. Repeat until the assign step no longer results in a change.

## The Code:

In this section we walk through each of the three steps of the algorithm, showing you how they can be constructed with ND4J's powerful NDArray data structure. 

### Step 1: Initialization of `k` centroids

There are several ways to initialize our first centroids, and we will use the Forgy method, which is to simply choose `k` random points from our actual data. 

In order to achieve our task of Forgy intitialization, we will first obtain the indices of k random points in the dataset we wish to cluster. In the below code our data will be contained in an `NDArray` called `data`. We will assume there are 3 clusters in the data.

```java
int k = 3
INDArray data = Nd4j.rand(10, 10);

Random r = new Random();
int[] meanIndexes = r.ints(k, 0, data.rows()).toArray();
centroids = data.getRows(meanIndexes);
``` 

We will track our clusters in a nested `List` called `clusters`. There will be `k` inner lists, each one representing a cluster. Each cluster will hold the integer indices of the data that belongs to it. Below we assign the first points to our clusters, namely our initial random data points from above:

```java
List<List<Integer>> clusters = new ArrayList<>();

for (int idx : meanIndexes) {
	ArrayList<Integer> cluster = new ArrayList<>();
	cluster.add(idx);
	clusters.add(cluster);
}	
```

Putting it all together in a simple method, we get:

```java

void init(int k, INDArray data) {
	Random r = new Random();
	int[] meanIndexes = r.ints(k, 0, data.rows()).toArray();
	centroids = data.getRows(meanIndexes);

	for (int idx : meanIndexes) {
		ArrayList<Integer> cluster = new ArrayList<>();
		cluster.add(idx);
		clusters.add(cluster);
	}
}	
```

### Step 2: Assignment
In the assignment step, we simply measure the Euclidean distance of each point to all centroids, and assign it to the corresponding closest cluster. We will find one particular method in ND4J's arsenal very useful here. Namely, we are going to use `Transforms.allEuclideanDistances`, to simplify this step. 

First, though, we need to track a few things:

1. The current point we are assigning, `currPoint`.
2. The distances to each centroid from the current point `distances`.
3. Whether or not a point has been moved from one cluster to another during the assignment step, `changed`. 
4. The closest centroid to the current point, `closest`.

```java
INDArray currPoint;
INDArray distances;
boolean changed = false;
int closest;
```
Then, in a loop we will find the closest centroid and assign that point to the corresponding cluster.

```java
for (int j = 0; j < data.rows(); j++) {
	currPoint = data.getRow(j).reshape(1, data.columns());
	distances = Transforms.allEuclideanDistances(currPoint, centroids, 1).mul(-1);
	closest = distances.argMax(1).getInt();

	if (!clusters.get(closest).contains(j)) {
		clusters.get(closest).add(j);
		changed = true;
	}
}
```
***Note:*** We multiply the resulting distances by -1 in order to take advantage of `argmax` to find the closest centroid.

***Note:*** We reshape the row (point) in our data we are considering so that the dimensions match our collection of centroids.

Again, we will put this together in a simple method:

```java
boolean assign(INDArray data) {
	INDArray currPoint;
	INDArray distances;
	boolean changed = false;
	int closest;

	for (int j = 0; j < data.rows(); j++) {
		currPoint = data.getRow(j).reshape(1, data.columns());
		distances = Transforms.allEuclideanDistances(currPoint, centroids, 1).mul(-1);
		closest = distances.argMax(1).getInt();

		if (!clusters.get(closest).contains(j)) {
			clusters.get(closest).add(j);
			changed = true;
		}
	}
	return changed;
}

```

### Step 3: Update

Now that we have every datapoint assigned to a cluster, we must update the centroids, which are the means values of each cluster. 

```java
void update(INDArray data) {

	// For each of k clusters
	int i = 0;
	for (List<Integer> cluster : clusters) {
		// get an array of indexes of points assigned to this cluster.
		int[] idxs = cluster.stream().mapToInt(Integer::valueOf).toArray();

		//calculate mean of this cluster's points.
		INDArray mean = data.getRows(idxs).mean(0);

		// update this cluster's centroid.
		centroids.putRow(i, mean);
		i++;
	}
}
```

## Putting it Together

With these three simple methods in place, we can execute K-means clustering in a final method we will call `fit`:

```java
void fit(INDArray data) {

	if (k != 0) {
		init(k, data); 				
		boolean changed = true;
		while (changed) {
			changed = assign(data); 
			update(data);			
		}
	} else {
		System.out.println("Please use the setK() method before running fit().");
	}
}

```

## Things to note

K-means is an effective clustering strategy for many applications. However, one of its main downfalls is that it is sensitive to the initial choice of centroids. The choice of centroids will determine the local minimum found by the algorithm, so in practice several runs are required, the best of which kept. 

One such implementation, called k-means++, provides an initial seeding method which places an upper bound on the within-cluster sum of squares (WCSS, a.k.a cluster variance). 

For further information, you may find https://en.wikipedia.org/wiki/K-means%2B%2B of interest. 


## Possible next steps

- Try implementing a way to run your algorithm multiple times, capturing the best results. Remember, K-means is subject to falling into local minima with sometimes very bad results!
- Try implementing spectral clustering: https://en.wikipedia.org/wiki/Spectral_clustering

## Runnable code

Below is runnable code, written for the purposes of this tutorial. It requires that you download the prototypical iris dataset and place it at the root of a project containing the below java class. Run it and enjoy a colored visualization of your clusters courtesy of X-Chart. 

```java
package clustering;

import org.knowm.xchart.SwingWrapper;
import org.knowm.xchart.XYChart;
import org.knowm.xchart.XYChartBuilder;
import org.knowm.xchart.XYSeries;
import org.nd4j.linalg.api.ndarray.INDArray;
import org.nd4j.linalg.factory.Nd4j;
import org.nd4j.linalg.ops.transforms.Transforms;
import tech.tablesaw.api.DoubleColumn;
import tech.tablesaw.api.Table;
import tech.tablesaw.columns.Column;
import tech.tablesaw.io.csv.CsvReadOptions;

import java.io.IOException;
import java.util.ArrayList;
import java.util.List;
import java.util.Random;

public class KMeans extends KClustering {

	private int k;
	private INDArray centroids = null;
	private List<List<Integer>> clusters;

	public KMeans() {
		this.clusters = new ArrayList<>();
	}

	/**
	 * This method is where the K-means algorithm runs.
	 *
	 * <p>Classic K-means runs in three steps: Initialize k centroids, assign data to those k centroids, update the
	 * value of those k-medoids. In this implementation, those three steps are the init, assign and update methods.
	 *
	 * @param data The data to fit K-means to.
	 */
	public void fit(INDArray data) {

		if (k != 0) {
			init(k, data);

			boolean changed = true;
			while (changed) {
				changed = assign(data);
				update(data);
			}

		} else {
			System.out.println("Please use the setK() method before running fit().");
		}
	}

	/**
	 * Forgy method of initial selection of k-means.
	 *
	 * @param k    The number of means to initialize.
	 * @param data The data from which random means will be selected.
	 */
	private void init(int k, INDArray data) {
		Random r = new Random();
		int[] meanIndexes = r.ints(k, 0, data.rows()).toArray();
		centroids = data.getRows(meanIndexes);

		for (int idx : meanIndexes) {
			ArrayList<Integer> cluster = new ArrayList<>();
			cluster.add(idx);
			clusters.add(cluster);
		}
	}

	/**
	 * Perform the assignment step of K-Means clustering.
	 *
	 * <p>For each point in the dataset, finds the closest centroid and assigns that data point
	 * to the corresponding cluster.
	 *
	 * @param data The data to cluster.
	 * @return whether or not an update to the current clusters was made.
	 */
	private boolean assign(INDArray data) {
		INDArray currPoint;
		INDArray distances;
		boolean changed = false;
		int closest;

		for (int j = 0; j < data.rows(); j++) {
			currPoint = data.getRow(j).reshape(1, data.columns());
			distances = Transforms.allEuclideanDistances(currPoint, centroids, 1).mul(-1);
			closest = distances.argMax(1).getInt();

			if (!clusters.get(closest).contains(j)) {
				clusters.get(closest).add(j);
				changed = true;
			}
		}
		return changed;
	}

	/**
	 * Performs the centroid update step of K-Means.
	 *
	 * <p>Calculates new centroids based on assignment of new points to a cluster.
	 *
	 * @param data The dataset being clustered.
	 */
	private void update(INDArray data) {

		// Iterate over the points assigned (by index) to a given cluster.
		int i = 0;
		for (List<Integer> cluster : clusters) {
			// get an array of indexes of points assigned to this cluster.
			int[] idxs = cluster.stream().mapToInt(Integer::valueOf).toArray();

			//calculate mean of this cluster's points.
			INDArray mean = data.getRows(idxs).mean(0);

			// update this cluster's centroid.
			centroids.putRow(i, mean);
			i++;
		}
	}

	/**
	 * This method sets the value of k for the first time or updates it. If the model has
	 * already been fit, setting a new k will clear the existing clusters and the model must
	 * be re-fit.
	 *
	 * @param k
	 */
	public void setK(int k) {
		clusters.clear();
		this.k = k;
	}

	/**
	 * Returns the model's current clusters based on last fit.
	 *
	 * @return A list of lists, where each of k inner lists contains the indices of the data that
	 * belongs to it.
	 */
	@Override
	public List<List<Integer>> getClusters() {
		return this.clusters;
	}

	/**
	 * Compute the nearest centroid or medoid based on pre-fit internal model.
	 *
	 * @param data The data to cluster.
	 */
	@Override
	public void cluster(INDArray data) {
		//TODO Check if fit was performed.
		//TODO If yes, compute correct cluster for each data point;
		//TODO
	}

	public static void main(String[] args) {

		// --------------- Read in CSV Data -------------//
		Table df = null;
		String path = "iris.data";

		CsvReadOptions options =
			CsvReadOptions.builder(path)
				.separator(',')
				.header(false)
				.build();

		try {
			df = Table.read().usingOptions(options);
		} catch (IOException e) {
			e.printStackTrace();
		}

		// ------ Separate independent and dependent variables -----//
		Column<String> labels = (Column<String>) df.column("C4");
		df.removeColumns("C4");
		double[][] data = df.as().doubleMatrix();

		// ----------- Get independent data into Ndarray ----------//
		INDArray input = Nd4j.createFromArray(data);

		// --------------------- Run K-means ----------------------//
		KMeans km = new KMeans();
		km.setK(3);
		km.fit(input);

		// ------------ Append Predictions as New Column ---------//
		DoubleColumn preds = DoubleColumn.create("Predictions", df.rowCount());

		int i = 0;
		for (List<Integer> cluster : km.clusters) {
			int[] idxs = cluster.stream().mapToInt(Integer::valueOf).toArray();
			for (int ix : idxs) {
				preds.set(ix, i);
			}
			i++;
		}
		df.addColumns(labels, preds);
		System.out.println(df.structure());
		System.out.println(df);


		// ------------------- Visualize Results -------------------- //
		Table type;
		XYChart chart = new XYChartBuilder().width(1200).height(800).build();
		chart.getStyler().setDefaultSeriesRenderStyle(XYSeries.XYSeriesRenderStyle.Scatter);


		for (int j = 0; j < 3; j++) {

			type = df.where(
				df.doubleColumn("Predictions").isEqualTo(j)
			);

			double[] xData = type.doubleColumn(2).asDoubleArray();
			double[] yData = type.doubleColumn(3).asDoubleArray();

			String seriesName = String.format("Classification %d", j);
			chart.addSeries(seriesName, xData, yData);
		}

		new SwingWrapper<>(chart).displayChart();
	}

}


```