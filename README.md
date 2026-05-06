# Audio Identification & Source Detection System

## Team Information
- **Team Name**: Sushimushi
- **Year**: 2026
- **All-Female Team**: No

## Architecture Overview

#### Describe your approach here. Keep it short and clear.

   ## Architecture Overview

#### Describe your approach here. Keep it short and clear.

    - [Our system extracts MFCC (Mel-Frequency Cepstral Coefficients) features using librosa — 13 coefficients per frame across the full audio file, flattened into a 1D float array per song. These arrays are stored in-memory in a FeatureStore dictionary keyed by integer song_id, enabling O(1) access. To enable fast lookup, each float feature is quantized to an integer hash and inserted into an InvertedIndex — a dictionary mapping hash → list of song_ids. This avoids loading or re-processing audio at query time.]
    - [We use a two-stage matching pipeline. First, the InvertedIndex narrows the search space: the query snippet's quantized hashes are looked up and candidate songs are ranked by number of hash hits using a Counter. The top 10 candidates are then scored using cosine similarity between the query's MFCC array and each candidate's stored feature array. For noisy queries, we apply fuzzy matching — both arrays are clipped to equal length and slight gaussian noise is added to the candidate before comparison, improving robustness to distortion and time offsets. The highest scoring candidate above a 0.15 confidence threshold is returned as the match.]
    - [The architecture is fully in-memory with no external database, making it lightweight and fast to spin up. The FeatureStore and InvertedIndex use Python dicts with threading.RLock on write operations, making reads concurrent and safe. Query processing is handled by a ThreadPoolExecutor with 4 workers, allowing multiple queries to be identified simultaneously without blocking. The system scales to thousands of songs since the InvertedIndex reduces per-query comparisons from O(N) brute force to O(top-K) where K=10.]
    - [Latency is minimized at two levels: the InvertedIndex lookup is O(1) per hash, reducing candidate set size before any similarity computation; and cosine similarity over fixed-length numpy arrays is vectorized and fast. End-to-end latency is tracked per query using a LatencyTracker with a rolling 100-query log. Accuracy is maintained despite noise through fuzzy cosine matching and a strict confidence threshold of 0.15 that filters out weak matches, preventing false positives on unknown or heavily distorted audio.]
    
 

**Note:** Please do not change the format or spelling of anything in this README. The fields are extracted using a script, so any changes to the structure or formatting may break the extraction process.
