<img width="200" alt="image" src="https://github.com/rttle/Bank-Churn-Kaggle-Challenge/assets/143844181/dbbeb760-7ac3-4d53-84ce-a08071725da1">

# LLM Project: Generating Pros and Cons List from Reviews 
This repository holds an attempt to fine tune an LLM to give a pros and cons list when given a watch product name, the data used is provided by McAuley Lab at the University of California San Diego (“UCSD”): https://amazon-reviews-2023.github.io/. 

## Overview
This project takes a dataset of Amazon reviews filtered for watch/watch-related products and prepares it as input for an LLM to generate pros and cons lists based on the products’ reviews. The project involves using one LLM to generate a pros and cons list to be used as the “true” label for evaluation. A second LLM is fined-tuned on a dataset that compiles the reviews by product with a column made up of the “true” label. Once the second LLM is fine-tuned, the first LLM is used to generate similarity scores, 1 to 10 where 1 is unrelated and 10 is the content of the two lists are the same. When plotted, the similarity scores show that the fine-tuned LLM does fairly well when the input is structured similar to the data; however, when you try to get the fine-tuned LLM to recall reviews by only giving the product name it does worse.

## Summary of Workdone
### Data
- Data: 
  - Input:  Product Name, Product Reviews, Generated Pros and Cons list (from ground truth LLM: GPT-4)
- Size: 158808 products
  - Used 500 products for training/validation
- Instances (Train, Validation Split): 
  - Train: 350 Products (70%)
  - Validation: 150 Products (30%)

### Preprocessing / Clean up

**Filtering Dataset.** The basis for the dataset used to train the LLM was the “Clothing_Shoes_and_Jewelry” and “meta_Clothing_Shoes_and_Jewelry” datasets from an Amazon Reviews scrapping project done by UCSD. Those datasets are very large, so to have a workable dataset size, it was decided to filter and keep only watches. Filtering was done by checking the lowest level of the product category for the text “watch.” This ended up capturing products that are watch-related. This result was accepted because the goal was to take a subset of the data that was more cohesive to make it easier for the LLM to determine positive and negative aspects of watches/watch-related products.

The image below shows how the categories related to a given product was structured as lists, which is why filtering was done based on the lowest level (last item in list).
<img width="780" height="81" alt="Screenshot 2025-08-09 at 9 21 28 PM" src="https://github.com/user-attachments/assets/b3e35d57-26a2-42ef-807a-b122199c17ed" />


**Aggregating** The result of the filtering was a file where each row was made up of a review for a product along with its meta data (product information). To make sure the LLM was getting enough information, the data was grouped by product, and the individual reviews of each product was compiled as lists. 

**Labeling** GPT-4 was used as the ground truth for this project; it was used to generate pros and cons list for each product in the prepared dataset. These lists were produced by prompting GPT-4 to extract a pros and cons list when given the product’s name and the compiled reviews. This process requires an Open AI API key that is linked to a funded account. The output from GPT-4, the pros and cons lists, were returned as a list, which was then concatenated onto the prepared dataset. The dataset after this portion of preprocessing/preparation is the final dataset to be used in fine-tuning GPT-3.5 for this project.

The figure below, is the first five rows of the final dataset to be used for fine-tuning. Note, this is before implementing the required formatting needed for fine-tuning; this is showcasing the data to be shown to the model.
<img width="916" height="138" alt="Screenshot 2025-08-09 at 9 25 16 PM" src="https://github.com/user-attachments/assets/4023ae4c-4696-4894-9d55-5bfcbc3367e1" />


### Problem Formulation
- Input / Output
  - Input: Product Name and Product Reviews from Amazon
  - Output: Pros and Cons List
- Models 
  - GPT 4 Turbo (Ground Truth/metrics)
  - GPT 3.5 = model trained/fine-tuned
- Hyperparameters
  - Epochs = {10}
  - Batches = {10}
  - Learning Rate = {0.0001}

### Training (Fine-tuning)
To fine-tune GPT-3.5 to take a user’s input of a product name to generate a pros and cons list for that product, the prepared dataset needed to be further formatted for fine-tuning. In particular, each row or instance had to be formatted to tell the LLM what it is meant to do, what the user will input is, and what the LLM should be generating in response. For this project, the LLM was told it will summarize the product’s reviews into a pros and cons list. The user will provide the product’s name and the compiled reviews. The LLM will then generate a pros and cons list.
Once the file for fine-tuning is properly formatted, the sample of 500 products was used to fine-tune GPT-3.5. Of the 500 products, 350 were designated as training and the other 150 was designated as validation. A fairly small sample size was used due to monetary limitations. The specific GPT-3.5 model fine-tuned was GPT-3.5 Turbo 1106, with batch size of 10, learning rate of 0.0001, and 10 epochs. Hyperparameter training was not used for this project to minimize the amount of money spent.



### Performance Comparison
The fine-tuned GPT-3.5 was tested in two ways. The first followed the training format and had the model take the validation set and generate pros and cons list when given the product name and reviews. The second asked through the prompt for the model to recall reviews for the product the user specified in their input and then generate a pros and cons list. The metrics used to judge the tests where similarity scores with a 1 to 10 system. These scores where graphed as histograms to show the distribution and visually show how well the model was doing in both situations. The similarity scores were generated by GPT-4, by prompting that model to take the two pros and cons list and give it a numerical score based on how similar their contents are.

Below is an example of a product’s “true” label (GPT-4.5) and the one generated by the fine-tuned GPT-3.5. The GPT-4.5 one has better formatting; however, the fine-tuned GPT-3.5 pros and cons list still had the same content.
<img width="743" height="409" alt="Screenshot 2025-08-09 at 10 18 15 PM" src="https://github.com/user-attachments/assets/d4edc6d2-d804-4c33-96c3-2d4662a78ef5" />

Below is an exmaple comparison when the fine-tuned GPT-3.5 was asked to recall reviews and only given a product's name.
<img width="769" height="471" alt="Screenshot 2025-08-09 at 10 20 12 PM" src="https://github.com/user-attachments/assets/fd121d79-1c77-4160-bb76-ec431e719908" />

When following the formatting of the training, the model did well with a mean similarity score of 8.98. The visualization also showed a left skew, meaning that the model was typically generating similar pros and cons list to the GPT-4 results.

When the model was asked to recall reviews from the training about the products to generate the pros and cons list, it did worse with a mean similarity score of 3.03. This was a poor choice, given that these products were from the validation set, so the model did not actually have reviews to recall. To correct this mistake, the same test was run again on a subset of the training set; however, the mean similarity score was also poor at 2.90.

Below are histograms of the similarity scores when testing was done using the validation dataset.
<img width="1240" height="470" alt="image" src="https://github.com/user-attachments/assets/bc67ff6e-23b8-4c8d-b290-03f15b0a2b96" />


Below is the histogram of the similarity scores when using a subset of the training dataset.
<img width="715" height="455" alt="image" src="https://github.com/user-attachments/assets/d88537cb-de1d-4884-8285-d41198a63bb1" />


### Conclusions
The fine-tuned GPT-3.5 did well when the user input matches the training input, product plus compiled reviews; while it unsurprisingly did worse when asked to recall reviews and then generate a pros and cons list when given a product name.

### Future Work
Rethinking the structure of the fine-tuning dataset could lead to better results when considering how we want users to interact with the LLM. In particular, a potential fix is to have duplicate rows where one has all the information (product, reviews, label) and a duplicate where it only has product and label. This would help the model learn it cannot just rely on the user’s input to generate its response and hopefully help train it to recall information better when prompted. This change would require editing of the process used to produce the formatted file for fine-tuning since the system’s role and user input would change. 
Once the model gets better at generating results based on user giving only a product name, it could then be expanded upon by scraping more reviews from other sources to give the model more context to pull from. This repository does include a notebook that shows how one watch forum was scraped for reviews; however, scraping data often involves significant preprocessing before use, so it was not further explored for this project due to time constraints. Ideally if more time could be dedicated, multiple sources could be scraped to get reviews from watch enthusiasts and then preprocess the scraped data to be able to use for fine-tuning. A major issue that would need to be address is how to pull out a product’s name from a review. This issue further extends when one also considers that some watches will have a formal official name along with a nickname like how one variant of the Rolex GMT-Master is called the Pepsi GMT or how the Seiko SBGA211 is called the Snowflake.

## How to reproduce results
To reproduce results, download the Clothing_Shoes_and_Jewelry User Review and Metadata jsonl.gz files from the linked repository. Then ensure that the LLM_Preprocessing.py file is downloaded from this repository and run the LLM_Labeling_FineTuning.ipynb notebook also found in this repository to recreate the fine-tuned GPT-3.5 shown in this project. 

## Overview of files in repository
- **alt_dataset_amazon.ipynb:** Notebook that takes the dataset and prepares it for modeling. Filtered the amazon dataset to only include watch or watch-related products.
- **LLM_Labeling_FineTuning.ipynb:** Notebook that takes the preprocessed dataset and makes use of GPT-4 to add labels to the dataset, which is then used to fine-tune GPT-3.5 to generate pros and cons list for products. Also includes visualization of results.
- **LLM_Preprocessing.py:** Module created to wrap all preprocessing done to the dataset in the alt_dataset_amazon notebook.
- **wus_scraper.ipynb:** Notebook that shows successful scraped a watch forum for reviews. Resulted in very noisy data.

## Data
Data is from McAuley Lab at the University of California San Diego. https://amazon-reviews-2023.github.io/

## Citations
- @article{hou2024bridging,
  title={Bridging Language and Items for Retrieval and Recommendation},
  author={Hou, Yupeng and Li, Jiacheng and He, Zhankui and Yan, An and Chen, Xiusi and McAuley, Julian},
  journal={arXiv preprint arXiv:2403.03952},
  year={2024}
}
WatchUSeek, *WatchUSeek Reviews Forum*, WATCHUSEEK.COM, https://www.watchuseek.com/forums/reviews.67/ 

