# Metrics for event detection in app reviews

This replication package contains the python scripts and data files used for the data science master's degree thesis of Leandra Moonsammy, with supervisor Quim Motger. 

## Structure

This package is structured in 3 main folders:

- **data** - Contains the data set of reviews and annotated truth sets used for experimentation.
- **scripts** - Contains the python scripts to apply the automatic processes depicted in our approach, mainly (1) preprocessing reviews, (2) the extraction of metrics, (3) automatically detecting events from mobile app reviews, and (4) the selection of reviews for summarization.
- **results** - Contains the output files generated by the previous automated processes.

## Dataset

We used the dataset from the paper [Unveiling Competition Dynamics in Mobile App Markets through User Reviews](https://arxiv.org/abs/2312.01981) by Motger et al. This paper selected a subset of microblogging apps which included Twitter, Mastodon, and 10 additional microblogging Android mobile apps based on user crowdsourced software recommendations from the [AlternativeTo platform](https://alternativeto.net/software/twitter/?platform=android). For each app, Motger et al. collected all reviews available in multiple repositories published within a time window of 52 weeks (~1 year), from June 9th, 2022 to June 7th, 2023 (included). From the complete list of 12 microblogging apps, we excluded 2 apps for which the number of available reviews was insufficient for statistical analysis. The complete list of apps and the number of available reviews is reported in the following table.

| App name      | App package                 | #reviews |
|---------------|-----------------------------|----------|
| X (Twitter)   | com.twitter.android         | 168,335  |
| Truth Social  | com.truthsocial.android.app | 4,580    |
| VK            | com.vkontakte.android       | 3,677    |
| Hive Social   | org.hiveinc.TheHive.android | 3,054    |
| GETTR         | com.gettr.gettr             | 2,592    |
| MeWe          | com.mewe                    | 2,660    |
| Mastodon      | org.joinmastodon.android    | 1,441    |
| Bluesky       | xyz.blueskyweb.app          | 625      |
| Minds         | com.minds.mobile            | 423      |
| CounterSocial | counter.social.android      | 252      |

The complete data set of reviews is available at ```data/microblogging-review-set.json```.

## Experimentation

In this section, we provide a detailed step-by-step description of the experimentation process using the scripts available in the ```scripts``` folder.

- Install the Python module requirements:
    
    ```pip install -r scripts/requirements.txt```

### Data preprocessing

- Preprocess the raw user reviews by providing the dataset and output folder. Note that the JSON file has to be unzipped first. 

  ```python ./scripts/preprocess_reviews.py -i data/microblogging-review-set.json -o data```

  The preprocessed data files are saved in the ```data``` folder. 

### Metric computation
    
- Compute the metrics for the all reviews truth set and the positive truth set. By default, we use a time window of size ```w = 7``` and the truth set timeframe of ```t = Oct 06, 2022 - Nov 30, 2022```. Note that the JSON files has to be unzipped first.
- All ratings metrics:

    ```python ./scripts/compute_metrics.py -i results/preprocessed_data/preprocessed_review_set.json -w 7 -t 'Oct 06, 2022 - Nov 30, 2022' -o results/metrics/all```

- Positive only metrics:

    ```python scripts/compute_metrics.py -i results/preprocessed_data/preprocessed_review_set_positive.json -w 7 -t 'Oct 06, 2022 - Nov 30, 2022' -o results/metrics/positive```

- To compute the metrics for inference, the same structure can be followed using the time frame of ```t = Dec 01, 2022 - Jan 25, 2023```.

### Event detection

After metric extraction, events can be automatically detected and classified as an event or non-event. The script requires the metrics folder for train/test and inference, and the truth set file. 
- All ratings:

  ```python ./scripts/predict_events.py -m results/metrics/all -t data/truth_set_all.csv -i results/metrics/all_inference```

- Positive  only:
  
  ```python ./scripts/predict_events.py -m results/metrics/positive -t data/truth_set_pos.csv -i results/metrics/positive_inference```

### Summarization (utilized from the replication package of Motger et al.)

To conduct a sample summarization of a selected time window:

- Use any reported event to extract a sample representation of the reviews published for a given app within a time window. The script requires specification of the following parameters: the app package (```-a```), the date interval (```-d```), the output folder (```-o```) and the number of reviews to collect in the sample (```-n```). 
	
    The folowing example collects a sample of 50 reviews from the Twitter app published between October 27th, 2022 and November 2nd, 2022 (i.e., the week when Twitter formally announced the buyout from Musk).

	```python ./scripts/get_reviews_by_date.py -i ./data/microblogging-review-set.json -a com.twitter.android -d 'Oct 27, 2022 - Nov 02, 2022' -o results -n 50```

- An output file containing the sample set of reviews is generated using the name template ```<a>-reviews-<d>.csv``` where ```<a>``` is the app package and ```d``` is the date interval. For instance, for the previous example, the following file will be generated:

	```com.twitter.android-reviews-Oct 27, 2022 - Nov 02, 2022```
    
- Use Claude AI to request a summarization of the most relevant events highlighted in the sample set of reviews. To do so, we used a prompt engineering approach within a zero shot learning context, for which the following prompt was designed:

	```
    <review-set>
    
    Identify and summarize the most significant event raised by this set of reviews extracted from mobile app repositories.
    ```
    
    Where ```<review-set>``` is the content of the sample review file generated in step #7.
    
	(!) Please note that systematic repetitions of this process can lead to slightly different results, mainly because of (1) the random selection of a sample set of reviews, and (2) the internal behaviour of Claude AI.
