1001 Tracklists Scraper
=======================
A set of functions to scrape music tracklists from [1001 Tracklists](https://www.1001tracklists.com)


Learning Ideas
--------------
1. NLP style learner
	+ Predicting high-cardinality single variable (track ID)
	+ Simple to implement via Fastai
	+ Sensitive to time domain series data
	- How are new tracks incorporated? Only uses what it knows.
	- New track scores would require similarity function that is programmed, not learned.
2. Rossman Dataset style learner
	+ Could predict feature(s), not just ID
	+ Sensitive to series data in the time domain
	- Not taylored for recommendations - similarity score between desired features and track options would have to be programmed.
	- Predicting multiple features may be difficult with stock Fastai library.
3. Collaborative Filtering style learner
	+ Easily assigns scores to unseen tracks (this is what it was made for)
	+ Good at incorporating many features as input (when in NN form, not simple matrix)
	- Potentially not as sequence-oriented; maybe fixed by adding time/index in set? perhaps add feature diffs from last song as features?
	- Requires re-thinking "user" field in predictor -> assign to "context" meaning previous set of tracks?
