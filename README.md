# Item Recommendation

Item recommendation is a recommendation system that determines which target words have the priority to be used in the
exercises. We have 16k target works a.k.a lemposes in Elia (lempos = lemma + pos, e.g. work_NOUN). However, in any moment some of the words 
will have higher probability to be forgotten by the users. Therefore, the lemposes practiced by the users have a probability of forgetting 
value calculated by a machine learning algorithm. This task is about creating a function that can run this ML model for each user and
each lempos(words) that they have seen. Hence, it should be optimised as much as possible. 

## Existing Method: Half-life regression

The method is defined [here](https://research.duolingo.com/papers/settles.acl16.pdf). It creates a feature vector for each user
to obtain half-life and the probability of recall for the given word. For example, if the half-time is 2 days, 2 days after the user
is practiced the word, the user has 0.5 chance to recall the word, in other words 1-0.5=0.5 is the probability of forgetting the word. 
Therefore, with the help of this method, a function will produce a probability of forgetting for each word practiced by the users. 
The words will be sorted with respect to the probabilities in descending order to create exercises. 

### Feature Vector

    lexeme_id: lempos (e.g. work_NOUN)
    timestamp: int --- Last time practiced 
    user_id: str 
    history_seen: --- total no of times user see the word 
    history_correct: --- total no of times user was right 
    wrong: int 
    session_correct: int
    session_seen: int
    
Please refer to duolingo_spaced_algo.py and [paper](https://research.duolingo.com/papers/settles.acl16.pdf) for detailed explanation
of features.


### Update for each user (with old DB Architecture)

<div align="center">
  <img src="lempos_com.png" width="300px" height="150px" />
  <p>User Interaction and Knowledge Tables</p>
</div>



Inputs to Algorithm:

    all rows of learner_info_history table including given user_id: log_id, info_id, date, knowledge
    
    log_id: unique id of sessions
    info_id: unique id of the information practiced (translation, synonym, form, etc of information of the lempos)
    date: datetime 
    knowledge: int enumerated as
    [ACT : 1, NRCG: 2, NTC :3, RCG: 4, NRCL: 5, PRCL: 6, RCL: 7]
    
    NRCL, NRCG -> incorrect answers
    RCG, PRCL, RCL -> correct answers
    NTC, ACT -> neither correct nor incorrect, it should be counted to the times that word is seen. 
    
##### Steps:
    1: Group by log_id to determine the sessions.
    2: Compute correct and incorrect answers for each session (NRL, NRCG-> incorrect - PRCL, RCL, RCG->correct).
    3: Compute total incorrect and correct answers for each row (NRL, NRCG-> incorrect - PRCL, RCL, RCG->correct).
    4: Join with component_information and lempos_component table to obtain lemposes.
    5. Group by lemposes to obtain latest date practiced.
    6. Compose feature vector for Spacing Algorithm.
    7: Retrive previous half-life.
    8. Update half-life and forgetting probability with Spacing method with new feature vector.


### Update for each user (new DB Architecture)

Instead of repeating the same computation to obtain the correct and incorrect answers for each lempos, 
create learner_component_history and learner_lempos_history to count correct and incorrect answers and
store the last date. 
