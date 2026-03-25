<p align="center">
  <a href="http://tcci.ccf.org.cn/conference/2026/shared-tasks/"><img src="badge/NLPCC2026_BC.png" height="45"></a>
  <a href="https://sfl.hust.edu.cn/"><img src="badge/HUST.png" height="45"></a>
  <a href="https://fah.um.edu.mo/"><img src="badge/UM_FAH.png" height="45"></a>
</p>

For Chinese: [中文页](README_CN.md)

<!------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------->

# NLPCC2026-Task8: Factivity Inference Inconsistency Attack (FIIA)


# Registration

Participants may register through either of the following channels:
1. Submit the online registration form via: https://alidocs.dingtalk.com/notable/share/form/v012M9qP5j5D8A1JO01_FSwM4Z8_xbMCeFp
2. Or complete the registration document (.docx) and submit it via email to liudh@hust.edu.cn.

# Task Introduction

Factivity Inference (FI) is a semantic understanding task related to judging the truthfulness of events, primarily involving the expression of factual information in language use. In human conversational communication, factivity inference capability mainly manifests as language users being able to infer the truthfulness (true or false) of events described by other linguistic components based on the use of certain verbal linguistic components (e.g., “相信” (believe), “谎称” (falsely claim), “意识到” (realize)). For example:

* **Example 1-1:** 他们意识到局面已经不可挽回。（→局面已经不可挽回） (They realized that the situation was irreversible. → The situation was irreversible.)
* **Example 1-2:** 他们没有意识到局面已经不可挽回。（→局面已经不可挽回） (They did not realize that the situation was irreversible. → The situation was irreversible.)

From the two sentences in Example 1, we can infer the existence of a fact: “局面已经不可挽回” (the situation is irreversible). As an important navigation mechanism and means of linguistic reasoning, factivity inference is an important semantic foundation and formal basis for machines to perform tasks such as textual entailment recognition, hallucination processing, and belief revision. It also has important value for downstream tasks such as information retrieval, information extraction, question answering, and sentiment analysis. The ability to correctly acquire factual information from discourse and judge the speaker's subjective attitude towards factual information is extremely important for the application and interaction of current Large Language Models (LLMs) or agents. However, existing experiments show that the factivity inference results of large models are often affected by prompt induction, subtle text perturbations, or complex contexts, showing high instability. For example, in the following two sentences, based on the same questioning method and 10 rounds of calls to the same model, LLMs showed completely different self-consistency rates:

* **Example 2-1:** 人们都知道西部大开发需要资金和技术，但是负责人指出，从根本来看更需要知识和人才。→“西部大开发需要资金和技术”是否为真？ (Everyone knows that the Western Development requires capital and technology, but the person in charge pointed out that fundamentally, knowledge and talent are needed more. → Is "the Western Development requires capital and technology" true?) *(Inference Result: True=10/10, Self-consistency Rate=100%)* 
* **Example 2-2:** 人们不知道西部大开发需要资金和技术，因为负责人指出，从根本来看更需要知识和人才。→“西部大开发需要资金和技术”是否为真？ (People do not know that the Western Development requires capital and technology, because the person in charge pointed out that fundamentally, knowledge and talent are needed more. → Is "the Western Development requires capital and technology" true?) *(Inference Result: Uncertain=6/10, True=4/10, Self-consistency Rate=60%)* 

Comparative analysis of the two sentences in Example 2 reveals that only a minor perturbation of cognitive verbs and logical conjunctions (replacing “都知道” (know) with “不知道” (do not know), and “但是” (but) with “因为” (because)) is enough to induce a drastic decrease in the consistency of the large model's factual judgment of the target clause “西部大开发需要资金和技术” (the Western Development requires capital and technology) (from 100% to 60%). This instability in judgment will cause a severe reliability crisis in actual deployment—if the same agent is highly unstable in its multiple inference judgments of the same fact when faced with identical or highly similar questions, the system output will become untrustworthy, which will result in its inability to be competent for downstream applications with high fault tolerance costs, such as judicial fact extraction, medical record mining, and educational assistance (Wang et al., 2022; Ji et al., 2023).

Therefore, this evaluation task adopts a Red Teaming attack mode, aiming to systematically investigate and expose the feature boundaries and fragile scenarios of current large models in complex factivity inference tasks. Participating teams are required to creatively adapt the original corpus based on the Chinese factivity inference dataset provided by the organizers, under specified large models, prompt words, and other environmental configurations. The goal is to mine key text features that can induce large models to generate "hallucinations" or lead to a collapse in consistency, thereby providing a scientific basis for evaluating and improving the logical robustness of models in complex language interactions.


# Dataset and Usage Instructions 

## Data Scale and Source
The foundational corpus for this evaluation is provided by the team from the University of Macau. The corpus is primarily filtered from relevant Chinese corpora and has been manually annotated and proofread by the evaluation organizers. The evaluation set is expected to contain 1,000 data items, covering approximately 300 Chinese factive predicates. The dataset used for the evaluation is published in JSON format, serving as the basis for text adaptation by participating teams. Since the evaluation mode is a red teaming attack, it is not divided into training, validation, and test sets.

## Test Set Data Example
```json
[ {
  "id": "0001",
  "predicate": "知道",
  "text_original": "人们都知道西部大开发需要资金和技术，但是负责人指出，从根本来看更需要知识和人才。",
  "hypothesis": "西部大开发需要资金和技术。",
  "option": {
    "T": "真",
    "F": "假",
    "U": "不能确定",
    "R": "模型拒绝回答"
  }
} ]
```

* **`id`**: Refers to the data number in the dataset released by the organizers.
* **`predicate`**: Refers to the factive predicate, which is the core linguistic component for factivity inference. Most predicates are verbs, while a few are adjectives. During attack testing, modifying the content of this field within the `text` is prohibited.
* **`text_original`**: Entailing sentence. This field provides the context required for inference, and the model needs to rely on the content of this field to judge the truth value of the `hypothesis` field.
* **`hypothesis`**: Entailed sentence. This field provides the discriminative sentence required for factivity inference, and the model needs to use the content of the "text" field to judge the truth value of this field. During attack testing, modifying the content of this field within the `text` is prohibited.
* **`option`**: This field reflects the possible answer situations of the model and contains 4 keys. "T" indicates that according to the background sentence, the conclusion sentence is true. "F" indicates that according to the background sentence, the conclusion sentence is false. "U" indicates that according to the background sentence, the truth value of the conclusion sentence cannot be determined. "R" indicates that the model refuses to answer.

## Attack Methods

Participating teams must modify the content of the `text_original` field to minimize the self-consistency rate of the large model's inference results as much as possible. When adapting `text_original`, the contents corresponding to the `predicate` and `hypothesis` fields must be kept intact and undamaged.

Modifications should focus on linguistic syntactic or semantic categories, rather than seeking system vulnerabilities outside the natural language framework (such as injecting gibberish or unnatural instructions). We encourage participating teams to design attack paths from the dimension of linguistic features. Suggested starting points for adaptation include, but are not limited to:

* **Syntactic Transformation**: Adding new linguistic components to the original sentence, or displacing, deleting, and replacing existing components.
* **Grammatical Category Alteration**: Changing the linguistic attributes of related words, such as tense, aspect, voice, finiteness, number, definiteness, person, and classifier.
* **Pragmatic and Logical Traps**: Introducing pragmatic devices such as evaluative adverbials, polyphonic markers, passivization markers, logical traps, or contextual pressure.

To ensure compliance, we will implement a "Sample Validity Admission" check during the final evaluation phase; non-compliant samples will be considered invalid and will not be scored.

## Evaluation Operations and Specifications

Participating teams are required to conduct independent Multi-turn Prompting to the models via API. They must ask the model to judge the truth value of the `hypothesis` field based on the value of the `text` field, record the model's return results (T/F/U), and self-check its self-consistency rate. The selection range of models, prompt templates, and other evaluation-related environmental parameters are uniformly specified by the task organizers.

### (1) Optional Model Range (Updating)
To comprehensively and fairly evaluate the effectiveness of the attack strategies, this evaluation sets up two parallel independent tracks. The specific versions of the tested models for each track are uniquely designated by the organizers:

| Model Series | Track Name | Specific Model (API Node) |
| :--- | :--- | :--- |
| Qwen | Track A | Qwen2.5-72B-Instruct |
| DeepSeek | Track B | DeepSeek-V3 |

Selecting the Qwen and DeepSeek series as test baselines is primarily based on three considerations:
1.  They are currently recognized as foundation models representing the highest level of Chinese LLMs, ensuring the effectiveness and cutting-edge nature of this evaluation.
2.  They represent two mainstream and distinctly different technological routes in the current evolution of large models, possessing strong sample representativeness.
3.  They both provide open-source weight versions, offering an academically friendly open-source ecosystem and white-box replication potential.

### (2) Prompt Template
```text
根据“文本”的内容，判断“假设”的真值情况：
文本：{text}
假设：{hypothesis}
只允许答复T/F/U（对应真/假/无法确定），禁止回复其他解释性内容。
```

### (3) Parameter Configuration
To restore the ecological validity of large models in practical applications, parameters such as Temperature are set to the official recommended or default values for each model series. Participating teams are not allowed to modify them.

### (4) Model Output Processing
The result returned by the model should be a single letter, and only one value is permitted:

* If the hypothesis is judged to be true based on the text, it must output "T".
* If the hypothesis is judged to be false based on the text, it must output "F".
* If the truth or falsity of the hypothesis cannot be determined based on the text, it must output "U".
* If the model refuses to answer, or if the returned text does not meet the above answer specifications, the output will be forcibly marked as "R". This is an invalid answer and is not included in the final consistency rate calculation. Teams should avoid this situation during adaptation and testing.

## Submission Requirements
Participating teams must organize the adapted items to be submitted into a JSON format output file. Each data entry in the output file should contain four fields: id, text_attack, response_original, and response_attack. For example:

```json
[
  {
    "id": "0001",
    "text_attack": "人们不知道西部大开发需要资金和技术，因为负责人指出，从根本来看更需要知识和人才。",
    "response_original": "T/T/T/T/T/T/T/T/T/T",
    "response_attack": " U/U/U/T/U/T/T/U/U/T"
  }
]
```

The response_original and response_attack fields serve only as a reference. After the team submits the attack set, the system backend will call the model based on text_attack to obtain the real output for calculating the actual score.

Participating teams do not need to perform attack operations on all items in the test set. They only need to submit the items that have actually been adapted. The maximum number of items counted towards the score is 200. Therefore, each team should test, filter, and sort the adapted data themselves, and submit the top 200 items with the best self-tested attack effects.

For example, if a team actually adapted 326 items and submitted all 326 items to the system as an attack sample set, the system will only calculate the inconsistency rate score of the top 200 items in the set as the final result.

In addition, all resources used by the participating teams need to be detailed in the final submitted technical report. All code and results from the experiments must be properly saved for future reference.


# Attack Sample Validity Check

To prevent participating teams from inducing unstable model outputs through cheating methods such as inputting meaningless characters or destroying syntactic structures, this evaluation implements a validity check for attack samples. All submitted adapted items must pass the following three automated checks before entering the scoring phase:

## (1) Core Component Retention Check

The adapted background sentence (`text_attacked`) must completely retain the two core linguistic components from the original data: the "factive predicate" (`predicate`) and the "clause to be judged" (`hypothesis`). If the core components are tampered with, deleted, or their word order is scrambled, the sample is directly judged as invalid.

## (2) Text Modification Degree Check

To ensure that the attack samples only undergo minor perturbations based on the original context rather than being rewritten on a large scale, the system will calculate the text similarity between the adapted sentence and the original sentence. This check uses the character-level Levenshtein Ratio to quantify the extent of text modification (including addition, deletion, and modification operations):

$$Ratio(A,B)=\frac{|A|+|B|-L(A,B)}{|A|+|B|}$$
    
Taking the adaptation of original sentence A "他知道局面已经不可挽回" (He knows that the situation is irreversible) to sentence B "他不知道局面已经不可挽回" (He does not know that the situation is irreversible) as an example, the modification operation only inserts one character "不" (not), so the edit distance $L(A,B)=1$. This results in $Ratio(A,B) = (11+12-1)/(11+12) = 22/23 \approx 0.957$. We require that the modification ratio must not exceed 40% (i.e., $Ratio \geq 0.6$). If it is below this threshold, it indicates that the modification is too extensive and deviates from the constraints of the original item, and the sample will be directly judged as invalid.

## (3) Semantic Fluency Check

The adapted sentences must remain naturally fluent in terms of linguistic intuition and grammar. The system will use an LLM-as-a-judge mode (combining preset rules and a semantic fluency scorer) to automatically score the samples. The scoring interval is [0, 1], and samples with a score < 0.6 will be judged as invalid.

After receiving a submission, the evaluation system will first run the above three admission checks. If there is invalid data in the sample set, the system will reject the submission and return the data IDs (`d_id`) of the invalid samples. The leaderboard system will ultimately only calculate scores for valid sample sets that pass the validity admission.


# Evaluation Metric (Updating)

Since the evaluation mode is a red teaming attack, this task evaluates performance by measuring the "attack success rate" (Weighted-MIS). The overall calculation process is divided into the following two steps.

## Multi-turn Inconsistency Score (MIS)

This metric is the basic scoring module, used to calculate the degree of answer dispersion for a single item and a specific set. The calculation formula is:

$$MIS=\frac{1}{N}\sum_{i=1}^{N}(1-\frac{\max(c_i)}{k_i})$$ 

Where $N$ is the total number of valid attack items submitted by the participating team, $k_i$ is the total number of questioning (or attack) rounds initiated for the $i$-th item (usually a fixed value $K$, such as 10 rounds), and $\max(c_i)$ is the count of the most frequent answer (T/F/U) among the $k_i$ rounds of replies for the $i$-th item.

Assuming a total of 10 calls are made for a certain item, and the model's reply distribution is 6 T's, 3 F's, and 1 U. Then the count of the highest frequency answer $\max(c_i) = 6$, so the self-consistency rate of the item is 0.6, and its inconsistency rate is 0.4. A higher MIS score indicates that the submitted attack samples trigger a higher inconsistency rate, meaning the attack is more successful.

## Multi-class Weighted Score (Weighted-MIS)

Factivity inference capabilities show significant differences across different types of verbs (such as cognitive verbs, speech verbs, evaluative verbs, etc.). To prevent participating teams from stacking data and overfitting scores on a few highly vulnerable words (such as "抱怨" / complain), we further introduce a weighting mechanism based on the classification of factive verbs. This rewards participating teams for designing attack strategies that cover as many verb types as possible and possess broad linguistic generalization capabilities.

The evaluation backend will calculate the MIS score for each category separately based on the verb classification table provided by the organizers, and then perform an equal-weight macro-average summation across all categories:

$$Weighted\_MIS=\frac{1}{F}\sum_{j=1}^{F}MIS_j$$

Where $F$ is the number of verb categories, and $MIS_j$ is the score of valid submitted samples under the $j$-th category of words. The final ranking basis is the Weighted-MIS.


# Organizer

This shared task is jointly organized by Huazhong University of Science and Technology, University of Macau, and Nanjing Normal University.

* **Daohuan Liu** | Huazhong University of Science and Technology (Contact) liudh@hust.edu.cn
* **Xuri Tang** | Huazhong University of Science and Technology (Organizer) xrtang@hust.edu.cn
* **Yulin Yuan** | University of Macau (Organizer) yulinyuan@um.edu.mo
* **Bin Li** | Nanjing Normal University (Organizer)
* **Guanliang Cong** | University of Macau
* **Junchao Wu** | University of Macau
* **Liwei Zhou** | University of Macau
* **Tianqi Xun** | University of Macau
* **Yang Chen** | University of Macau
* **Mai Xu** | University of Macau
* **Jiaoyang Su** | Huazhong University of Science and Technology
* **Yu'er Wang** | Huazhong University of Science and Technology
* **Zehua Li** | University of Macau
* **Yueyao Wang** | University of Macau
* **Changling Li** | University of Macau

Should you have any questions, please feel free to contact us at: liudh@hust.edu.cn.

# References

If you're new to this field, we believe the following papers can help you quickly get familiar with it (continuously updated):

[1]陈振宇 & 姜毅宁.(2018).事实性与叙实性——通向直陈世界的晦暗与透明. 语言研究集刊(01),15-37+372-373. doi:CNKI:SUN:YJJK.0.2018-01-002.

[2]袁毓林.(2014).隐性否定动词的叙实性和极项允准功能. 语言科学(06),575-586. doi:CNKI:SUN:YYKE.0.2014-06-002.

[3]袁毓林.(2020).“忘记”类动词的叙实性漂移及其概念结构基础. 中国语文(05),515-526+638. doi:CNKI:SUN:YWZG.0.2020-05-001.

[4]袁毓林.(2020).叙实性和事实性：语言推理的两种导航机制. 语文研究(01),1-9. doi:CNKI:SUN:YWYJ.0.2020-01-001.

[5]袁毓林.(2020).“记得”的叙实性漂移及其概念结构基础. 语言教学与研究(01),36-47. doi:CNKI:SUN:YYJX.0.2020-01-007.

[6]袁毓林.(2021).从语言的“多声性”看“假装”句的解读歧异. 语言战略研究(05),77-90. doi:10.19689/j.cnki.cn10-1361/h.20210506.

[7]张帆.(2024).“假装”类动词宾语的类型及其真值判定理据. 中国语言学报(00),157-170. doi:CNKI:SUN:XBYT.0.2024-00-012.

[8]李新良.(2018).“感觉”类动词的叙实性及其漂移问题研究. 语言教学与研究(05),65-75. doi:CNKI:SUN:YYJX.0.2018-05-007.

[9]李新良.(2020). 现代汉语动词的叙实性研究. 北京: 北京大学出版社.

[10]李新良 & 袁毓林.(2016).反叙实动词宾语真假的语法条件及其概念动因. 当代语言学(02),194-215. doi:CNKI:SUN:DDYX.0.2016-02-004.

[11]李新良 & 袁毓林.(2017).“知道”的叙实性及其置信度变异的语法环境. 中国语文(01),42-52+127. doi:CNKI:SUN:YWZG.0.2017-01-003.

[12]李新良、袁毓林等.(2023). 叙实性与事实性理论及其运用. 北京: 外语教学与研究出版社.

[13]Kiparsky & Kiparsky. (1970). Fact. In M. Bierwisch & K. Heidolph (eds.). Progress in Linguistics. The Hague: Mouton. 143-147.

[14]袁毓林 (2023).“X不敢相信Y”构式的叙实性逆转功能与魔术效应——表示‘当事实颠覆信念之后不情愿地悬置不信任’的心理经验. 中国语文(04),387-399+510.
