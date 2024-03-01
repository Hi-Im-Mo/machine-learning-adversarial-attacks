# Introduction to Adversarial Attacks in NLP


## Project Objectives:
* Understand & implement the basics of Natural Language Processing (NLP) 
* Describe the use-case differences between pre-trained and trained NLP models
* Generate adversarial NLP examples to probe and strengthen on our NLP model
* Importance in securing ML and AI models
* Future direction of adversarial application in LLM research

## Project Intro
Our project can be divided into 4 main components:
1. Data Extraction, Tranformation & Loading
2. Model Training & Evalutation
3. Adversarial Attack 
4. Model Optimization

Each stage of the project has an accompanying anotated ipykernel notebook. The project is designed to be a self-contained workflow where all a user needs to do is clone the repository to their local device & launch the repository in Jupyter notebook or Google collab. In a future update, I hope to push skeleton versions of the scripts to make this project easier to learn from and apply. 

## Getting Started
1. Clone this repo (for help see this [tutorial](https://help.github.com/articles/cloning-a-repository/)).
2. 'scraper.ipynb' contains the webscraper program. The data has already been saved in a shared MongoDB cluster & the password is currently pushed to the repository. If this project gains traction, I would love to make the data easier to access in a cloud format or maybe an S3 bucket and turn the scraper notebook into a .py program. 
3. 'models.ipynb' contains the scripts for the NLP models used in training & evaluation. The cells are arranged to introduce increasing levels of NLP classification complexity, with the pretrained VADER Sentiment Intensity Analyzer pipeline introduced and compared at the end of the notebook. 
4. 'unstemmed_attack.ipynb' contains the scripts for the initial adversarial attack on our dataset. 
5. 'stemmed_attack.ipynb' contains the scripts for the optimal preprocessing of our text data to make it more resilient to adversarial attack examples.

## Methods
1. Data ETL

The first dilemma when it came to training our own NLP model was the lack of text data readily available. After exploring various retail websites with reviews for their products, we landed on Walmart as a source of a plethora of unstructred text data with a clear, standardized labeling system already in place. The 'scraper' program utilizes the open source package 'splinter' to call a webdriver instance and directs the webdriver to the product reviews URL. It then pulls the HTML of the entire webpage and saves it into a variable. Next, the program calls on the 'BeautifulSoup' open source parsing package to parse the HTML string variable and extract the 'date', 'text', and 'stars' of each rating in on the page. The dictionary holdiing this data is appended to a master DataFrame object and then the program sleeps itself for a random number of seconds between 110 and 140 to simulate human activity on the website. There was some success utilizing a range of 70 to 90 seconds, but after a duration of time, Walmart's algorithm began to prompt the scraper to verify it was a human. After the data is compiled, duplicates are dropped, the DataFrame is processed back into a dictionary and inserted into a Mongo database entitled 'walmart' and a collection entitled 'scraper'. The program prints its progress to the cell terminal for tracking. 

2. Model Training & Evalutation

The 'models.ipynb' file in the forefront of the repository contains the scripts for model training & optimization. For this introduction to NLP, we utilize scikit-learn pipelines to maintain consistency across increasing model classification complexity and make the topic easier to understand from a beginner point-of-view. All of the pipeline objects tokenize and vectorize the data in term frequency times the inverse document frequency (TF-IDF). The binary sentiment classifier pipeline utilizes a naive Bayes classifier algorithm due to the high level of testing accuracy demonstrated coupled with the low computational complexity of the model. In subsequent classification tasks, the algorithm changes to a stochastic gradient descent method for optimization. As mentioned before, the naive Bayes model classifies NLP sentiment based on a binary label, either positive (1) or negative (0). The model that follows that example classifies data on a polarity range, labeling text as either positive (1), neutral (0), or negative (-1). The data utilized to train this model is preprocessed somewhat simply, classifying all 4- & 5-star reviews as positive sentiment, all 3-star reviews as neutral sentiment, and 1- and 2-star reviews as negative sentiment. In a future update, I want to utilize VADER's SIA model to classify the text data on the polarity range, label the data with VADER's prediction, and observe if this improves the classification accuracy in testing. The final model attempts to capture the most variance from label-to-label in the data as it classifies text examples based on how many stars the review would get in a range of 1-5. The data is preprocessed intuitively and fed to a pipeline utilizing the same vectorizer and SGD method of classification. Each model iteration indicates decreasing accuracy in testing; however, utilizing a confusion matrix, we can tell that the data is still learning to capture variation in the data. In other words, although testing accuracy in decreasing, the models are still capturing a lot of the variation in text sentiment and making *close* predictions to the actual label. We demonstrate this classification ability by introducing 6 examples of textual sentiment that one can intuitively classify and see what the model predicts. In every model, the classifier does a great job a predicting the label of the text. In the last model, we consider utilizing the VADER pretrained sentiment classifier. VADER represents an entirely different method of tokenize and engineering significant features for the model to understand semantic context and grammatical sentence structure, plus its already been pretrained on a massive amount of social media text data that has been preprocessed in the complex aforementioned method. This makes the VADER an excellent choice for general purpose sentiment text classification, but makes it fall short to our trained NLP models because it has been trained on an enormous *different* datast. 

3. Adversarial Attack

The 'unstemmed_attack.ipynb' file contains the scripts for the start of the generative adversarial efforts. In this notebook, we retrained a naive Bayes classifier on the binary preprocessed data. The pipeline looks different in this file, but accomplishes the same thing as the scikit-learn pipeline utilized in the 'models.ipynb' notebook. The purpose of this is that we need a variable containing the TF-IDF vectors of the text data to feed to the open source 'textattack' adversarial attack API library utilized in the generation of adversarial examples for our Walmart data. So, the pipeline from before is decomposed into a hard-wired version of transform textual tokens into TF-IDF vectors. In order to assemble the model into the textattack adversarial architecture, the model has to be contained within a special wrapper for scikit-learn models. After the model is wrapped, a pretrained adversarial attacker model is called and fitted on the same Walmart dataset as the trained NLP model. This pretrained attacker comes from the empirical validated scholarly journal article 'Is BERT Really Robust? A Strong Baseline for Natural Language Attack on Text Classification and Entailment' from Jin et al (2019) that has already been trained on sentiment text data to create transformation attacks on the scikit-learn model and attempt to successfully cause the model to misclassify without the transformation being noticed. Our trained model is observed to be highly affected by this attacker model even after testing a range of training data lengths for the NLP model. 

4. Model Optimization
   
The 'stemmed_attack.ipynb' contains the scripts for the advanced preprocessing method intended to defend the trained NLP classifier from *TextFooler* transformation attacks. 'Stemming' the dataset refers to utilize an algorithm for classifying the individual characters that compose a word before tokenization. Fortunately, the Natural Language Toolkit from from Python's base libraries contains a tool to encode a string this exact way. The PorterStemmer() function takes a string a classifies individual consonants as 'c', individual vowels as 'v', subsequent consonants as 'C', and subsequent vowels as 'V'. This encoding method strengthens the model against adversarial transformation attacks by giving the algorithm a method to dicern semantic changes in the data. Thus, when the adversarial model attempts to transform a word, the stemmed encoding of the word changes significantly and the model can recognize the attempt to change its training data. The improved resilience is demonstrated, and far greater when the length of training data in the set is maximized. 

## Discussion
So, in short, we found that stemming during tokenization gives the model more semantic understanding about words, which better shields our model from adversarial transformation attacks like we demonstrated earlier. Using the methods listed above, we succesfully caused a natural language processing model to go rogue and misclassify the sentiment of text examples, then we demonstrated a way to utilize adversarial attack models to learn about the robustness of a given model & strengthen its resilience to poisioned data. 

You might have the lingering question as to why it's important that we study adversarial attack on natural language processing when we already have large language models far more complex than text classifiers?

If we think about NLP models as the building blocks for far more complex systems like the chatbot & assistant models used by Alexa, Siri and ChatGPT, then it becomes vital that we understand the full capabilities and limitations of modular components of the system. When you build a stack of blocks, you want blocks to be strong and secure so that when the stack becomes taller, it does not collapse under the weakness of its blocks. Adversarial AI in NLP is an ever-expanding field and it is of peak importance that we understand its role in LLMs and RNNs with NLP components before shipping any of these complex systems into production use. Otherwise, they will still be vulnerable to attacks just like our unstemmed dataset could be easily transformed into ineffectual training data. 

## Contributors
|LinkedIn     |
|---------|
|[Karson Kosek](https://www.linkedin.com/in/karson-kosek-75981128a/)|
|[Morgan Escue](https://www.linkedin.com/in/morgan-martin-a41433186/) |
|[Forest Prodan](https://www.linkedin.com/in/forest-prodan-728957248/) |
|[Nargiz Shemmsulldin](https://www.linkedin.com/in/nargiz-shemssulldin-a115322a3/) |
