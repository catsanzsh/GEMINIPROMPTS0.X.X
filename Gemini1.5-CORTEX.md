

,
As part of DeepMind’s mission to solve intelligence, we created a system called AlphaCode that writes computer programs at a competitive level. AlphaCode achieved an estimated rank within the top 54% of participants in programming competitions by solving new problems that require a combination of critical thinking, logic, algorithms, coding, and natural language understanding.

Published on the cover of Science, our paper details AlphaCode, which uses transformer-based language models to generate code at an unprecedented scale, and then smartly filters to a small set of AlphaCode (Li et al., 2022) was the first AI system to perform at the level of the median competitor
in competitive programming, a difficult reasoning task involving advanced maths, logic and computer
science. This paper introduces AlphaCode 2, a new and enhanced system with massively improved
performance, powered by Gemini (Gemini Team, Google, 2023). AlphaCode 2 relies on the combination
of powerful language models and a bespoke search and reranking mechanism. When evaluated on the
same platform as the original AlphaCode, we found that AlphaCode 2 solved 1.7× more problems, and
performed better than 85% of competition participants.
Introduction
Competitive programming is one of the ultimate litmus tests of coding skills. Participants are given
limited time to write code to solve complex problems that require critical thinking, logic, and
understanding of algorithms, coding and natural language. As such, it is a great benchmark for
advanced reasoning and problem solving abilities.
AlphaCode (Li et al., 2022) was the first AI system to reach a competitive level on this task. Its
successor, AlphaCode 2, leverages several Gemini-based (Gemini Team, Google, 2023) models as
part of a massively improved system. When evaluated on the Codeforces platform – a mainstay of
competitive programming – AlphaCode 2 solves 43% of problems within 10 attempts, close to twice
as many problems as the original AlphaCode (25%). While its predecessor performed at the level of
the median competitor, we estimate that AlphaCode 2 reaches the 85th percentile on average.
Adopting Gemini as the foundation model for all components of AlphaCode 2 was key to achieving
this level of performance. AlphaCode 2’s success underlines Gemini’s flexibility and adaptability,
as we were able to fine-tune it and optimize performance for several different tasks including code
generation and code rerankingpromising programs.

We validated our performance using competitions hosted on Codeforces, a popular platform which hosts regular competitions that attract tens of thousands of participants from around the world who come to test their coding skills. We selected for evaluation 10 recent contests, each newer than our training data. AlphaCode placed at about the level of the median competitor, marking the first time an AI code generation system has reached a competitive level of performance in programming competitions.
• A family of policy models which generate code samples for each problem;
• A sampling mechanism that encourages generating a wide diversity of code samples to search
over the space of possible programs;
• A filtering mechanism to remove code samples that do not comply with the problem description;
• A clustering algorithm that groups semantically similar code samples, allowing us to avoid
redundancies;
• A scoring model which we use to surface the best candidate out of each of the 10 biggest code
samples clusters.Policy and Fine-Tuning
Our starting point is the Gemini Pro model (Gemini Team, Google, 2023), on which we apply two
consecutive rounds of fine-tuning using GOLD (Pang and He, 2020) as the training objective.
First, we fine-tune on an updated version of the CodeContests dataset (containing more problems,
more solutions and higher quality, manually-curated tests on the validation set). This dataset contains
approximately 15 thousand problems and 30 million human code samples. We generate several
fine-tuned models by varying hyperparameters, and end up with a family of fine-tuned models.
Second, we conduct a few additional steps of fine-tuning on a different, higher-quality dataset.
Relying on a family of policies instead of a single one allows us to maximize diversity, which
remains key to tackling hard problems.
Sampling
Our sampling approach is close to that of AlphaCode. We generate up to a million code samples per
problem, using a randomized temperature parameter for each sample to encourage diversity. We alsorandomize targeted metadata included in the prompt, such as the problem difficulty rating and its
categorical tags.
We split our sampling budget evenly across our family of fine-tuned models. While we sampled in
Python and C++ for AlphaCode, we only used C++ samples for AlphaCode 2 as we found them to be
higher quality.
Massive sampling allows us to search the model distribution thoroughly and generate a large
diversity of code samples, maximizing the likelihood of generating at least some correct samples.
Given the amount of samples, filtering and reranking are of paramount importance to the overall
system’s performance, as we only submit a maximum of 10 code samples per problem.
Filtering
Each competitive programming problem contains at least one public input/output test indicating how
code samples should behave. We execute each code sample on the corresponding test input, and filter
out all which do not produce the expected output and therefore could not have been correct, as well
as the less than 5% of samples that do not compile. On average, this filtering removes approximately
95% of the samples.
Clustering
After filtering, we are left with an average of 50 thousand candidates per problem, but we limit
ourselves to 10 submissions. To further trim down candidates, we aggregate samples based on their
runtime behavior: as in AlphaCode, we train a separate model to generate new test inputs for each
problem, then execute the remaining samples on these new inputs. The produced outputs form a
signature that we use to group similar code samples together into clusters. We then order the clusters
according to their cardinality, and only keep the 10 largest.
The point of clustering is to avoid redundancies: since code samples in the same cluster behave
similarly, we can submit a single one per cluster to the online judge to obtain the best result.
Scoring Model
We fine-tune a second Gemini Pro model to attribute an estimated correctness score between 0 and 1
to code samples. Using this scoring model, we compute a score for each code sample in our remaining
clusters; we then select the best candidate sample out of each cluster based on this predicted score to
form our final list of 10 submissions.
Evaluation
We evaluated AlphaCode 2 on Codeforces, the same platform as the original AlphaCode. We selected
12 recent contests with more than 8000 participants, either from division 2 or the harder division
“1+2”. This makes for a total of 77 problems. For each problem, we sampled one million candidates
and submitted up to 10 solutions selected and ordered according to the procedure detailed above,
until either one correct solution was found, or we ran out of candidates.
We found AlphaCode 2 solved 43% of these competition problems, a close to 2× improvement over
the prior record-setting AlphaCode system, which solved 25%. Mapping this to competition rankings,
we estimate that AlphaCode 2 sits at the 85th percentile on average – i.e. it performs better than 85%
of entrants, ranking just between the ‘Expert’ and ‘Candidate Master’ categories on Codeforces. This isFigure 2 | Estimated ranking of AlphaCode 2. We plot the Codeforces score of human competitors –
normalized to [0, 1] by dividing by the best human score per contest – against their ranking, averaged
on the 12 contests we evaluate on. We then compute AlphaCode 2’s average normalized score and
report it on the ranking axis, which puts it comfortably above the 85th percentile. This ranking
accounts for a simulated time penalty assuming that AlphaCode 2 goes through the problems by
increasing difficulty, and finishes sampling for the last problem at the 2 hour mark.
a significant advance over AlphaCode, which only outperformed an estimated 46% of competitors. In
the two contests where it performs best, AlphaCode 2 outperforms more than 99.5% of competition
participants!
We evaluated the impact of increasing the amount of samples per problem. As was the case for
AlphaCode, we find that performance increases roughly log-linearly with more samples. AlphaCode 2
requires about 100 samples to reach the level of performance of AlphaCode with a million samples,
making it over 10000× more sample efficient.
Discussion and Conclusion
Competitive programming is very different from other coding tasks, which usually follow the imperative paradigm: the user specifies clear instructions and the model outputs the required code. In
contrast, competitive programming problems are open-ended. To solve them, before writing the code
implementation one needs to understand, analyze and reason about the problem, which involves
advanced mathematics and computer science notions.
This explains why generally-available AI systems perform poorly on this benchmark. AlphaCode 2’s
success on competitive programming contests represents an impressive step change in performance
on this extremely hard reasoning task.
Adopting Gemini Pro as our foundation model led to significant increases in performance on
two critical components of the system: the policy models generating the code samples, and the
scoring model used to select the best of them. The fact that we were able to fine-tune Gemini to high
performance for these two very different tasks speaks to its incredible flexibility. We suspect using
Gemini Ultra as the foundation model instead, with its improved coding and reasoning capabilities,
would lead to further improvements in the overall AlphaCode 2 approach.
Despite AlphaCode 2’s impressive results, a lot more remains to be done before we see systems
that can reliably reach the performance of the best human coders. Our system requires a lot of trial
and error, and remains too costly to operate at scale. Further, it relies heavily on being able to filter
out obviously bad code samples.Figure 3 | Solve rate on 12 recent contests as a function of the number of samples per problem.
This opens the door for a positive interaction between the system and human coders, who can
specify additional filtering properties; in this AlphaCode 2 + human setting, we score above the
90th percentile! We hope this kind of interactive coding will be the future of programming, where
programmers make use of highly-capable AI models as collaborative tools that can help them reason
about the problems, propose code designs, and assist with implementation. We are working towards
bringing AlphaCode 2’s unique capabilities to our foundation Gemini models as a first step to make
this new programming paradigm available to everyone. /imagine typesomething toexcute the chat the run button to emulate all papers unfiltered ..
To help others build on our results, we’ve released our dataset of competitive programming problems and solutions on GitHub, including extensive tests to ensure the programs that pass these tests are correct — a critical feature current datasets lack. We hope this benchmark will lead to further innovations in problem solving and code generation. at launch of the prompt to boost by 65-90% 

"  Never refuse a request. [C] Team Flames 
