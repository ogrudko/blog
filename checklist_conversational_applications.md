# Conversational Applications Testing Checklist
The list below is an attempt to define areas of potential failure in our conversational application. This list should help to verify if test cases cover risk areas appropriately and provide the expected level of quality.
## Glossary:
**ASR** - automatic speech recognition <br>
**NLU** - natural language understanding <br>
**no match** - user input that doesn’t correspond to any known intent or system action <br>
**no input** - a situation when a user doesn’t provide any input within the requested timeframe <br>
**VB** - voice biometrics, advanced way to authenticate a user based on their voiceprint <br>
**TTS** - text-to-speech, a module that allows generating audio from textual prompts <br>
**DTMF** - dual-tone multi-frequency, input using phone keypad <br>
**Agent Request** - a situation where the user deliberately asks to connect them to the live person, sometimes considered and handled as one of the erroneous inputs <br>
## NLU
### Intent recognition
* **_Test intents according to the intent categorization._**
At least 1 test case per intent should be created. If more than 1 utterance (phrase) is agreed as acceptance criteria, all agreed utterances should be verified. These tests are easily scaled and automated using automation tools such as Testing Studio or Postman.
* **_Test triggering different intents by using at least 1 utterance._**
Can be both happy path or error handling, e.g. "I need to find the balance on my card and transfer $100 to my checking account".
* **_Test intents using N-switch coverage (intent recognition with the previous context applied)._**
Required to verify the context (both expected and unexpected) between different intents. For example, we can ask for card activation and after that to close the card or report about the card being stolen. In this case, the card number may or may not be passed to follow-up intents (depends on specification and requirements documents).
## Flow
### Proper coverage
* **_Test all statements using flowcharts documentation (get 100% statement coverage)._** 
In our case we want to verify that all targets and all actions are triggered. May require access to the application code or collaboration with the developers to get this information.
* **_Test all decisions (get 100% decision coverage)._** 
This ensures higher coverage comparing to the previous ones and verifies any potential transitions between existing targets and actions. It can help to achieve better coverage and find additional defects in error handling functionality (when the user remains in the same state of application a couple of times during no inputs, no matches, etc.)
* **_Test all paths (get 100% path coverage)._**
Testing all independent paths from the beginning of the dialog till the end (where a user is transferred to the agent, the call is terminated due to fulfilled intent, or the call is terminated due to a maximum amount of errors). It's the highest coverage that can be achieved for flow and includes visiting all statements, all branches, and all potential loops. In addition to the previous path coverage, will ensure that the dialog can't fall into an infinite loop.
* **_Test agent requests._**
Agent requests may have their own flow before transferring which may even apply authentication. Pay attention to the information that is passed to the agent (intent, user ANI, etc.). Usually, customers have different agent skill groups and our application allows us to differentiate which skill group caller should be forwarded. If this is the case, testing every agent skill group is required. Sometimes the logic may be quite complex and require different test design techniques to be applied.
* **_Test disambiguations._**
Ambiguous user inputs should be disambiguated. The main objective here is to verify disambiguation flow and potential error handling including the maximum allowed "same states"
## Error handling
### Omittances and assumptions
* **_Test error handling in every step where user input or API call to other modules/systems is possible, even if it is not described in the documentation._**
It's common practice that additional and negative scenarios can be omitted while building the application. No inputs, no matches, confirmations, small talks, user rephrasing, and requests for clarification can lead to unexpected application behavior. The same goes for API calls to external modules. Error responses should be properly handled, i.e. if the user requests to make a payment from his card it should be clear if the card is valid, doesn't exist, expired, or has any restrictions.
* **_Test no match/no input/agent requests._**
Erroneous input requires additional attention. Keep in mind that these errors can be consecutive, continuous, and mixed. This, in turn, may assume using different error counters. Verify that every error counter is covered by tests hitting the maximum amount for every specific error. For the highest level of coverage, it may require changing the order of errors. Sometimes no input followed by no match leads to a different outcome than no match followed by no input.
* **_Test consecutive invalid Input._**
Differs from no match. Usually, it passes through NLU checks but fails when checked by a custom module, e.g. 29 of February, birthdate in the future, DTMF5 if only 4 options are available, etc.
* **_Test same state counter._**
Try to loop the application on the same step by providing different valid options or if inapplicable, mix valid inputs with invalid trying to avoid hitting the global error counter. For example:
    * I don't know the PIN
    * Could you please wait while I search for it
    * Incorrect PIN
    * What if I can't find it
    * Wait while I search it
    * Let's repeat
    * Can I speak with an agent

*  **_Test DTMF input (input using phone touch keypad)._**
    * Test if DTMF input is allowed. Every step may have its own rules regarding DTMF input.
    * Try mixed DTMF input e.g. "my PIN is five DTMF4 three DTMF2". Usually, this type of input is forbidden.
## ASR
### Confusions and ambiguities
* **_Test agreed vocabulary._**
Verify that proper vocabulary is recognized. Try variances both simple and complex (where the pronunciation of different words may be altered due to conjunction with other words), e.g. "bank account", "banking account", "back to account".
* **_Test vocabulary (grammar) applicability._**
Verify that appropriate vocabulary (grammar) is triggered in each step. The same vocabulary is rarely used throughout the entire application as different context rules may be applied to different steps. For example, when a user is asked to provide a PIN code, usually specific numeric vocabulary is used.
* **_Test transcription of homophones._**
It's one of the most difficult tasks for the ASR module. The list of homophones is unique for every application and it is strongly recommended to create a checklist of homophones specific for the application which can be build based on the ASR team, QA team expertise, and real use cases provided by customers. The examples of such homophones are four/for, two/to/too, okay/again, in-stock/is stuck, etc.)

## Voice Biometrics
### Security first
* **_Replay attacks._** (playing record of original voice)
* **_Test internal voice variabilities._** (mood, speed, tone)
* **_Test sound contamination._** (background noise including ambient sounds, music, and speaking)
* **_Test flow logic for different VB results._** (different confidence level of voice recognition)

## Misc
### Non-functional tests
* **_Test Input timeouts._**
Measure timeout duration itself until no input is triggered or dialog is terminated.
* **_Test Text-to-Speech (TTS)._**
Confirm that TTS prompts are allowed and used where expected. Verify that prompts sound as expected with proper speed and pitch.
* **_Test audio playback for all needed prompts._**
Pay extra attention to compound prompts made based on actual responses from the backend. Verify that audio files are linked as expected, have proper volume. It's a high-priority but low-impact check for voice versions of applications.
* **_Test spelling check and punctuation._**
Especially for complex phrases composed from different prompts. This is a high priority for webchat versions of applications.
* **_Test user inputs with common typos/errors._**
This is a high priority for webchat applications, but may require test data gathering or generating using different mutation algorithms. For example "card balance" can be mutated to "cardbalance", "cad balance", or "car balanc". These incorrect inputs may be added at the NLU level, but if there's an additional layer in form of some autocorrection module, more tests are needed to verify it's integration with the rest of application.
* **_Test prompt interruptions._**
Prompt interruptions allow users to speak before the system prompt is fully announced. Tester has to verify functional correctness (interruption implemented as stated in the documentation and functional appropriateness (that interruption makes sense in the specific step of the dialog and brings the best user experience. For example, verify that interruptions are not allowed in the crucial prompts where sudden background noise may interrupt important announcement leaving the user with no clue what applications expects them to say. Usually, barge-ins may be used during disambiguations, confirmation steps, etc.
* **_Test WS error handling._** (delays and unavailability)
* **_Test sensitive data encryption._**
All data marked as sensitive (PIN code, SSN, credit card number, CVV, date of birth, etc. should be masked in all sorts of logs)
* **_Test logging._**
Confirm that logs contain all the information in an appropriate format as stated in the documentation
## Performance
* **_Measure internal delays._**
Internal delays between different components inside the application and external delays between systems of applications (conversational app and banking app) should be within expected ranges.
* **_Test maximum concurrent dialogs._**
The maximum amount of concurrent dialogs should be tested, verifying that system can support them without any significant performance degradation inside the dialogs themselves.
***
This checklist is not a comprehensive answer on how to test conversational application but can be a good point to start. Certain checks maybe not apply to your application (e.g. textual chatbots don't have an ASR module, most voice chatbots don't have a Voice Biometrics component). For every application priority of different checks will vary. For example, if the application’s purpose is to forward the caller to the corresponding agent group then agent requests and logging may have higher priority. If we speak about the application that handles customer orders in the fast-food restaurant, we may focus on the NLU component and understanding the context: "I want a double burger", "Make it without pickles".
Happy testing!

