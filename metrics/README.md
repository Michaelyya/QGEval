## How to use
Our codes provide the ability to `evaluate automatic metrics` which concludes the ability to `calculate automatic metrics`. Please follow these steps to calculate automatic QG metrics and evaluate automatic metrics on our benchmark.

You can use this code in two ways:
- Install the package, `pip install QGEval-metrics`. For usage, please refer to [https://pypi.org/project/QGEval-metrics/](https://pypi.org/project/QGEval-metrics/).
- Download this resource, and use it following the instructions below.

### Enviroment
run `pip install -r requirements.txt` to install the required packages.

### Calculate Automatic Metrics
1. Prepare data

    Use the data we provided at [../data/scores.xlsx](../data/scores.xlsx), or use your own data, which should provide passages, answers, and references.

2. Calculate automatic metrics. 
    - Download necessary models for each metric respectively. Model download sites can be found in each metric's link in [readme of QGEval](../README.md).
    - Update model path inside the codes. See `QRelScore` as an example.
        ```python
        # update the path of mlm_model and clm_model
        def corpus_qrel(preds, contexts, device='cuda'):
            assert len(contexts) == len(preds)
            mlm_model = 'model/bert-base-cased'
            clm_model = 'model/gpt2'
            scorer = QRelScore(mlm_model=mlm_model,
                    clm_model=clm_model,
                    batch_size=16,
                    nthreads=4,
                    device=device)
            scores = scorer.compute_score_flatten(contexts, preds)
            return scores
        ```

    - Run `python metrics.py` to calculate your assigned metrics results by changing `score_names` in `metrics.py`. (`data_path` in each file should be changed into your own data path)
        ```python
        # Run QRelScore and RQUGE based on our dataset
        # load data
        data_path = '../data/scores.xlsx'
        save_path = './result/metric_result.xlsx'
        data = pd.read_excel(data_path)
        hypos = data['prediction'].tolist()
        refs_list = [data['reference'].tolist()]
        contexts = data['passage'].tolist()
        answers = data['answer'].tolist()
        # scores to use
        score_names = ['QRelScore', 'RQUGE']

        # run metrics
        res = get_metrics(hypos, refs_list, contexts, answers, score_names=score_names)

        # handle results
        for k, v in res.items():
            data[k] = v
        print(data.columns)

        # save results
        data.to_excel(save_path, index=False)
        ```
    - or run the code file for specific metric to calculate. For example, run `python qrel.py` to calculate QRelScore results. The code file of each metric:
        ```python
        # metric: code file
        BLEU/ROUGE/METEOR/BERTScore: base_evaluator.py
        MoverScore: MoverScore.py
        BLEURT: test_bleurt.py
        Q-Metric: QBLEU/answerability_score.py
        QSTS: qsts.py
        BARTScore: BARTScore/bart_score.py
        GPTScore: gptscore.py
        UniEval: UniEval/unieval.py
        QRelScore: qrel.py
        RQUGE: RQUGE/rquge_score.py
        GPT-zeroshot: prompt_gpt.py
        G-Eval: geval.py
        KDA: 
        ```
    
### Evaluate Automatic Metrics
Run `python coeff.py` to obtain the Pearson, Spearman, and Kendall correlation coefficient between the generated results and the labeled results. For detailed process, please refer to [readme of QGEval](../README.md).
