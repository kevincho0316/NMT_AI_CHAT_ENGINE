nmt-chatbot
===================
이문서는 조재준에 의해 제작됨



Setup
-------------

 1. ```$ https://github.com/kevincho0316/NMT_AI_CHAT_ENGINE.git```
 2. ```$ cd nmt-chatbot```
 3. ```$ cd setup``` 
 4. ```$ python prepare_data.py``` ...Run setup/prepare_data.py - new folder called "data" will be created with prepared training data
 5. ```$ cd ../```
 6. ```$ python train.py``` Begin training
 7. (optional) you can run ```tensorboard --logdir=./model/``` at the project dir


Rules files
-------------

Setup folder contains multiple "rules" files (All of them are regex-based:

 - vocab_blacklist.txt - removes entities from vocab files
 - vocab_replace.txt - synonyms, replaces entity or it's part with replacement
 - answers_blacklist.txt - disallows answer from being returned (more on that later)
 - answers_detokenize.txt - detokenization rules (removes unneccessary spaces)
 - answers_replace - synonyms, replaces phrase or it's part with replacement
 - protected_phrases.txt - ensures that matching phrases will remain untouched when building vocab file




Tests
-------------

Every rules file has related test script. Those test scripts might be treated as some kind of unit testing. Every modification of fules files might be check against those tests but every modification should be also followed by new test cases in those scripts.

It's important to check everything before training new model. Even slight change might break something.

Test scripts will display check status, checked sentence and evetually check result (if diffrent than assumed).




More detailed informations about training a model
-------------

setup/settings.py consists of multiple settings:

 - untouched file/folder paths should fit for most cases
 - "preprocessing" dictionary should be easy to understand
 - "hparams" dictionary will be passed to NMT like commandline options with standard usage

setup/prepare_data.py:

 - walks thru files placed in "new_data" folder - train.(from|to), tst2012.(from|to)m tst2013(from|to)
 - tokenizes all sentences (adds spaces based on internal rules)
 - for "train" files - builds vocabulary files and checks entities against vocab_replace.txt rules, then vocab_blacklist_rules.txt, finally makes dictionary unique and saves up to number of entities set in setup/settings.py file, rest of entities will be saved to separate file

train.py - starts training process




Utils
-------------

utils/run_tensorboard.py is easy to use wrapper starting tensorboard with model folder

utils/pairing_testing_outputs.py - joins model/output_dev file with data/tst2012.form file and prints result to a console allowing easy check if things are going ok during training. Console will consist of input phrase, inferenced output frame and separator.



Inference
-------------

Whenever model is trained, inference.py, when direct called, allows to "talk to" AI in interactive mode. It will start and setup everythin needed to use trained model (using saved hparams file within model folder and setup/settings.py for the rest of settings or lack of hparams file).

For every question will be printed up to number of responses set in setup/settings.py. Every response will be marked with one of three colours:

 - green - first one with that colour is a candidate  to be returned. Answers with tah color passed clacklist check (setup/response_blacklist.txt)
 - orange - still proper responses, but doesn't pass check agains blacklist
 - red - inproper response - includes `<unk>`

Steps from the question to the answers:

 1. Pass question
 2. Compute up to number of responses set in setup/settings.py
 3. Detokenize answers using rules from setup/answers_detokenized.txt
 3. Replace responses or their parts using rules from setup/answers_replace.txt
 4. Score responses with -1 (includes `<unk>`, 0 (matches agains at least one rule in setup/answers_blacklist.txt file) or 1 (passes all checks)
 5. Return (show with interactive mode) responses

It is also possible to process batch of the questions by simply using command redirection:

    python inference.py < input_file

or:

    python inference.py < input_file > output_file

Importing nmt-chatbot
-------------

Project allows to be imported for the needs of inference. Simply embed folder within your project and import, then use:

    from nmt-chatbot.inference import inference

    print(inference("Hello!"))

inference() takes two parameters:

 - `question` (required)
 - `include_blacklisted = True` (optional)

For a single question, function will return dictionary containing:

 - answers - list of all answers
 - scores - list of scores for answers
 - best_index - index of best answer
 - best_score - score of best answer (-1, 0 or 1)

Score:

 - -1 - inproper response - includes `<unk>`
 - 0 - proper response but blacklisted
 - 1 - proper response - passed checks

With list of questions function will return list of dictionaries.

For every empty question function will return `None` instead of result dictionary.

With `include_blacklisted` set to false funtion will return either -1 or 1 for the score (and related to that score index)

made by재준
----------
"# NMT_AI_CHAT_ENGINE" 
