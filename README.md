# Information extraction from scholar figures
This project: downloads PDF's from Libgen, extract the images from PDF's, classifies them, and extracts their information.

## Part 1: Downloading the PDF's
DOI's from papers were extracted using Scopus.
The search in Scopus was with the following keywords:
("biodegradable film" OR "biodegradable composite" OR "nanocomposite film" OR "nanocomposite" OR
"bionanocomposite" OR "biocomposite"). The search was limited to papers in the areas of materials science, engineering, chemistry and chemical engineering. It was also limited to articles, conference papers or reviews. Finally, the search was sorted on relevance. It had a total of 104.457 documents. The DOI'S of the 2000 most relevant were exported into a TXT file.

Then, this file was used to download the papers from Libgen using [Scihub2pdf](https://github.com/bibcure/scihub2pdf).
(Sometimes Libgen would block my IP and stop the script. I had to wait one day to continue downloading).
Of, these, Scihub2pdf downloaded 1673 PDF's.

## Part 2: Extracting images from PDF's
I used [pdffigures2](https://github.com/allenai/pdffigures2).
Runned in Ubuntu. It's necessary to have Java and SBT installed. 
You may use [this tutorial](https://www.youtube.com/watch?v=uYcSYCGITeU) for the installation of SBT.

Once installed SBT, in the terminal you need to create a project, its name, and design it a folder. In terminal:
```
sbt new scala/scala-seed.g8
cd <project_name>
```

Then, go the folder you chose. There is a file called "build.sbt". Replace its content with:
```
import Dependencies._

ThisBuild / scalaVersion     := "2.13.1"
ThisBuild / version          := "0.1.0-SNAPSHOT"
ThisBuild / organization     := "com.example"
ThisBuild / organizationName := "example"

lazy val root = (project in file("."))
  .settings(
    name := "here appears the name you chose",
    libraryDependencies += scalaTest % Test,
    libraryDependencies += "org.allenai" %% "pdffigures2" % "0.0.11",
    scalaVersion := "2.11.11",
    resolvers += Resolver.bintrayRepo("allenai", "maven")
  )
```
Be careful with the "name" inside settings.

Now go to your terminal, change your directory to the project folder (the same directory that contains the build file). Then type: sbt
Wait for SBT to load, then type something like this:
```
runMain org.allenai.pdffigures2.FigureExtractorBatchCli /path/where/PDFsare -s stat_file.json -m /path/to/store/images -d /path/to/store/dataofimages
```
It's going to run and store the images in one folder and the captions and text in the other. This script has sometimes several problems.
When a problem occurs, it stops, nevertheless, it knows which PDF's have been analyzed. However, when you start it over, it analyzes those
already analyzed PDF's again. What I did was to eliminate the PDF's that generated errors (manually) and also, the PDF's already 
analyzed (with a script). This script goes like this:
```
#!/bin/bash
cd /path/to/store/dataofimages
ls > list.txt
sed -i 's/json/pdf/g' list.txt
mv /path/to/store/dataofimages/list.txt /path/where/PDFsare
cd /path/where/PDFsare
xargs rm -rf <list.txt
```
I executed the script for every error ocurred.

## Part 3: Building the deep learning model that classifies images in linear and non-linear
The deep learning model is developed to predict whether an image is linear or non-linear.

Two kind of DL models were generated: through manual optimization and through fine-tuning.

Three different notebooks were generated for this objective: the first one (INCEPTION-50epochs-10e-4) is an example of fine-tuning.
The second notebook (Clasificador_UU) contains the manual optimization method used, were -through for loops- different models were created. 
The last one is the template in which the models generated in notebooks 1 and 2 were tested (Test_sin_batches-plantilla).

After running many different combination of models (see 'informe_final' for more detail), the best one achieved a precission of 94%. This model can be downloaded from [here](https://drive.google.com/file/d/181uim1zvqu_IJqK98U0wt-kKS6vFbxx7/view?usp=sharing).

## Part 4: Detecting and extracting the edges, the curves and the metadata
This last step focuses on extracting information from the figures. First, the edges were detected. This was done by calculating an average of all of the column and row values from the image. This allowed to see rows or columns where black colors were predominant, and in this way, to the detect the X or Y ax. Later, to strenghten this process, a reward system was designed in order to avoid mistakes in figures where there were many black lines, or many edges or even in those where no axes existed.

Later, using a Python package the different curves (with different colors) in the image were identified and extracted, so the pixel position could be known. Also, once the X and Y axes were detected, their values were sent to a Google API that recognized text in pictures. Finally, having the values of the exes and the pixel positions of the curves and axes, allowed to interpolate and to build the original graph using Matplotlib.

This whole part 4 is shown in the jupyter notebook: "Information_line_extraction.ipynb".
