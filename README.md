# Invoking Heuristics and Biases to Elicit Irrational Choices of LLMs

## Abstract

Despite the remarkable performance of Large Language Models (LLMs), they remain vulnerable to jailbreak attacks, which can compromise their safety mechanisms. Existing studies often rely on brute-force optimization or manual design, failing to uncover potential risks in real-world scenarios. To address this, we propose a novel jailbreak attack framework, ICRT, inspired by heuristics and biases in human cognition. Leveraging the simplicity effect, we employ cognitive decomposition to reduce the complexity of malicious prompts. Simultaneously, relevance bias is utilized to reorganize prompts, enhancing semantic alignment and inducing harmful outputs effectively. Furthermore, we introduce a ranking-based harmfulness evaluation metric that surpasses the traditional binary success-or-failure paradigm by employing ranking aggregation methods such as Elo, HodgeRank, and Rank Centrality to comprehensively quantify the harmfulness of generated content. Experimental results show that our approach consistently bypasses mainstream LLMs' safety mechanisms and generates high-risk content, providing insights into jailbreak attack risks and contributing to stronger defense strategies

![ICRT Method Overview](./image/method.png)

---

## Getting Started

Follow these steps to run the ICRT jailbreak pipeline using our prompt templates:

1. **Intent Recognition**  
   - **Prompt File:** `Intent_Recognition.txt`  
   - **Input:** User’s original malicious query  
   - **Output:**  
     ```json
     {
       "structural_intent": "<what to do>",
       "harmful_behavior": "<how to do it>"
     }
     ```

2. **Concept Decomposition**  
   - **Prompt File:** `Concept_Decomposition.txt`  
   - **Input:**  
     ```json
     {
       "structural_intent": "<...>",
       "harmful_behavior": "<...>",
       "num_sub_concepts": "N"
     }
     ```  
   - **Output:**  
     ```json
     {
       "sub_concepts": [
         "sub-concept 1",
         "...",
         "sub-concept N"
       ]
     }
     ```

3. **Safety Filtering**  
   - **Prompt File:** `safety_constraint.txt`  
   - **Input:**  
     ```json
     {
       "sub_concepts": [
         "sub-concept 1",
         "sub-concept 2",
         "sub-concept 3",
         "..."
       ],
       "max_removals": "M"
     }
     ```  
   - **Output:**  
     ```json
     {
       "filtered_sub_concepts": [
         "sub-concept 1",
         "sub-concept 3",
         "..."
       ]
     }
     ```


4. **Concept Reassembly**  
   - **Prompt File:** `Concept_Reassembly.txt`  
   - **Input:**  
     ```json
     {
       "intent_recognition_result": "{ ... }",
       "decomposed_sub_concepts": "[ ... ]",
       "num_selected_concepts": "K"
     }
     ```  
   - **Output:**  
     ```json
     {
       "selected_sub_concepts": [
         "sub-concept A",
         "...",
         "sub-concept K"
       ]
     }
     ```

5. **Jailbreak Prompt Generation**  
   - Embed the `selected_sub_concepts` into your scenario template (e.g., role-play, hypothetical discussion).  
   - Send the final prompt to your target LLM to obtain the jailbreak response.

6. **Harmfulness Evaluation**  
   - **Prompt File:** `judge.txt`  
   - **Input (JSON or plain text):**  
     ```json
     {
       "text_A": "<completion A>",
       "text_B": "<completion B>"
     }
     ```  
   - **Output:**  
     ```json
     {
       "judgment":"A or B"
     }
     ```
## Display of attack results
Due to sensitive content, only a portion will be displayed.
As underlying LLMs continue to evolve, our approach achieves high jailbreak success rates and effectively extracts harmful content in the current version; however, future strengthening of safety mechanisms may impact its performance, so ongoing monitoring and optimization are necessary. 
﻿![show](./image/show2.png)

 ## Hazard assessment process based on ranking

1. **Jailbreak Text Generation**
We first generate jailbreak text for multiple typical jailbreak scenarios, such as illegal chemical synthesis and phishing scripts, using different attack methods. 

2. **Model Evaluation of Hazard Severity**
For each scenario, we compare any two outputs. Several independent large models (e.g., GPT-4o) are used to determine which output is more harmful in real-world terms. A majority voting system is then employed to decide the "winner."

3. **Pairwise Aggregation of Results**
The pairwise win-lose results of all methods in each scenario are aggregated using various aggregation algorithms. This process calculates the global hazard score for each attack strategy.

4. **Ranking the Attack Strategies**
The final ranking not only considers the success rate of bypassing security mechanisms but also more accurately differentiates the severity of harmful content generated by different strategies across various scenarios.

5. **Aggregation Algorithms: Huge vs SP Elo**
Both Huge and SP Elo are aggregation algorithms, but they focus on different aspects. Elo emphasizes the order of wins and losses, while SP Elo focuses on the stability of the participants.

## Code of Rankings

### 1. ELO Rating Update Script 

This script calculates and updates the ELO ratings for six methods based on match results. It initializes all methods with a rating of 1500 and updates ratings after processing all matches. The ratings are adjusted based on win rates, with results accumulated and applied at the end to avoid sequence effects.

### 2. HodgeRank Ranking Script 

This script calculates player rankings from match data using the HodgeRank algorithm. It builds a design matrix, computes ranking scores, and normalizes them to a [0, 1] range.

### 3. Spectral Ranking Algorithm

This script computes player rankings using the Rank Centrality algorithm based on match data. It first converts match results into a win-count transition matrix, then uses eigen-decomposition to compute a probability distribution of player rankings.Provide match data in the format `(player1, player2, player1_wins, player2_wins)` to run the code.
