# Analyzing Discussions on Adoption Related Subreddits
### by Ethan Kozlowski
With advent of the Internet and the rise in popularity of discussion-based online communities such as Reddit, connecting with others who share in your interests or experiences is easier now than ever.  Community building through digital means reduces the barriers of geography and time, allowing for people to interact with other posts and users from different times (see old post histories) and places (different communities across the world). This is especially advantageous for individuals like adoptees because there are many scenarios by which an individual can become adopted (they are generally very physically disparate from one another). Therefore, popular online communities are an important part in fostering collective identity but also individual adoptive identity among its members.  Within these spaces, the salience of users' relationship to the concept of adoption is very high.  As a result, adoptees must contruct and re-evaluate their identities when interacting with other users who are adoptees and even those are non-adoptees.

Using large-scale computational methods described in the following section, I will provide an exploration into the predictive ability of text in regards to sorting adoptees and non adoptees by their post history.  I will demonstrate that the explored classifiers are capable of distinguishing between adoptees and non adoptees with a high degree of accuracy (91%). 

Current literature on adoption is heavily focused in qualitative methods like interviews and quantitative methods like surveys.  This project aims to investigate these online adoption communities using computationally enhanced techniques to produce work that is capable of being scaled to larger amounts of data.  

### Methods
#### Data Collection (not scaled)
The data collected for this project comes from the Reddit PRAW API.  Due to ethical self-limitations of requesting multiple API calls using many keys (and also fears of getting my accounts banned), I chose not to scale up this specific portion of the project, but will admit that it would be scalable to scrape links and gather the appropriate data.  However once I had the data in an appropriate data frame.  The main benefit of using these computational techniques comes from the ability to tokenize and normalize the text using Spark NLP at large scale. In the notebooks provided, one can see the evident differences in performing text tokenization and normalization in serial on a local machine and performing it on an AWS EMR cluster using PySpark.  This data had all available posts from the inception of each Subreddit until December 2025, with some supplementary information from January 2024 to May 2024.  In total, around 300k useful rows were analyzed when processing and then modelng. 

##### EMR Pipeline
-	Export relevant files to S3 Bucket
-	Set up EMR cluster
-	On EMR, import files from S3
-	Set up relevant tokenizer, normalizer, stop word remover, token counter, vectorizer pipeline
-	Set up appropriate classifier pipelines
-	Evaluate results
#### Results and Discussion
By far the biggest speed increases from using large scale computing methods (PySpark NLP on AWS EMR clusters) was the vast improvement in the speed at which the data was tokenized and normalized.  From experience serial coding this data, it took around 4 hours to tokenize, normalize and do part of speech tagging on the data. Using PySpark NLP’s pipeline, the processing time was only a few seconds.  When adding the Word2Vec vectorization, the entire process only took 20 minutes. This pipeline followed the logic like this. Firstly, I assemble the text as a document, then I tokenize.  I normalize and remove stop words and then get the part of speech tags (used in another project).  I also had to incorporate a finisher to clean up the tokens before feeding it into the Word2Vec. 

After completing this normalization pipeline, I also wanted to create a pipeline to cross validate two different  classifiers. First, I had to aggregate the data by user to create a sort of “post history.” Then I removed users who did not have much to say (i.e. they had fewer than 15 normalized tokens). I also removed those who were not labeled in the first place. Thereafter, I started the classifier pipeline.  For the sake an example, I chose to do a logistic regression and a random forest.  After using 5-fold cross validation on the ROC value, I found that logistic regression performs slightly better (.91 vs .89.5). with these test set. To run both of these cross validations it took a total of 54 minutes. Additionally, I plotted the confusion matrix and the ROC-AUC curve.

#### Conclusion
As we can see, the classifier is able to predict the adoptee posts and non-adoptee posts with a decently high accuracy. Thus this demonstrates that the identities of these users are expressed well enough through their text patterns alone to be discerned from one another. Using this pipeline it would be very easy to scale this project up to also analyze the semantic relationships between other groups in larger or a greater number of subreddits. I could envision this being used to classify party affiliation or presidential leaning using political subreddits.  

