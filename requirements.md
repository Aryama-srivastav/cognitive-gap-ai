# Requirements Document: Cognitive Gap AI Learning System

## Introduction

The Cognitive Gap AI Learning System is an intelligent learning platform that detects why learners are stuck, tracks their accumulated knowledge debt, and validates true understanding through teach-back methodology. Unlike traditional AI tutoring systems that simply provide more explanations, this system diagnoses cognitive gaps, maintains a strategic learning memory, and ensures mastery through active validation. The system creates a closed learning loop where diagnosis, targeted learning, and validation work together to eliminate knowledge debt systematically.

## Glossary

- **System**: The Cognitive Gap AI Learning System
- **Learner**: A user interacting with the system to learn a subject
- **Knowledge_Debt_Graph**: A persistent data structure tracking all conceptual gaps, misconceptions, and learning priorities for a learner
- **Debt_Node**: A single entry in the Knowledge_Debt_Graph representing a specific knowledge gap or misconception
- **Cognitive_Gap**: A missing piece of understanding that prevents a learner from progressing
- **Diagnosis_Engine**: The component that analyzes why a learner is stuck
- **Teach_Back_Validator**: The component that evaluates a learner's explanation to validate understanding
- **Micro_Explanation**: A targeted, minimal explanation addressing a specific cognitive gap
- **Severity_Score**: A numerical value (0-100) indicating the priority of addressing a specific knowledge debt
- **Closed_Learning_Loop**: The complete cycle of diagnosis → explanation → teach-back → validation → debt update

## Requirements

### Requirement 1: Accept Learner Input

**User Story:** As a learner, I want to submit my current problem in multiple formats, so that I can get help regardless of how I express being stuck.

#### Acceptance Criteria

1. WHEN a learner submits code with an error, THE System SHALL accept and store the code and error message
2. WHEN a learner submits a natural language question, THE System SHALL accept and store the question text
3. WHEN a learner submits a raw "I'm stuck" statement without details, THE System SHALL accept and store the statement
4. WHEN a learner submits a combination of code and explanation, THE System SHALL accept and store both components
5. THE System SHALL validate that input is non-empty before processing

### Requirement 2: Diagnose Cognitive Gaps

**User Story:** As a learner, I want the system to identify WHY I'm stuck before explaining anything, so that I receive targeted help instead of generic explanations.

#### Acceptance Criteria

1. WHEN the Diagnosis_Engine receives learner input, THE System SHALL analyze the input to identify the root cause of confusion
2. WHEN analyzing input, THE System SHALL classify the gap as one of: conceptual gap, misconception, missing prerequisite, or cognitive overload
3. WHEN a diagnosis is complete, THE System SHALL generate a diagnosis statement explaining the root cause
4. THE System SHALL complete diagnosis before generating any explanatory content
5. WHEN the diagnosis is ambiguous, THE System SHALL ask clarifying questions before proceeding

### Requirement 3: Generate Targeted Micro-Explanations

**User Story:** As a learner, I want to receive minimal, focused explanations that address my specific gap, so that I'm not overwhelmed with unnecessary information.

#### Acceptance Criteria

1. WHEN a cognitive gap is diagnosed, THE System SHALL generate a Micro_Explanation addressing only that specific gap
2. THE System SHALL limit explanations to the minimum content necessary for understanding
3. WHEN generating explanations, THE System SHALL reference the diagnosed gap type in the explanation structure
4. THE System SHALL avoid explaining concepts the learner already understands based on Knowledge_Debt_Graph history

### Requirement 4: Initialize Knowledge Debt Graph

**User Story:** As a learner, I want the system to remember all my learning gaps across sessions, so that my learning is strategic and cumulative.

#### Acceptance Criteria

1. WHEN a learner first interacts with the System, THE System SHALL create an empty Knowledge_Debt_Graph for that learner
2. THE System SHALL persist the Knowledge_Debt_Graph across all sessions
3. WHEN creating a Knowledge_Debt_Graph, THE System SHALL assign a unique identifier to the learner
4. THE System SHALL initialize the graph with empty collections for Debt_Nodes

### Requirement 5: Track Knowledge Debt

**User Story:** As a learner, I want the system to track every gap, misconception, and weak understanding I demonstrate, so that I can see what I need to work on most.

#### Acceptance Criteria

1. WHEN a cognitive gap is diagnosed, THE System SHALL create or update a Debt_Node in the Knowledge_Debt_Graph
2. WHEN a learner repeats the same error, THE System SHALL increment the occurrence count for the corresponding Debt_Node
3. WHEN a learner avoids a topic across multiple sessions, THE System SHALL create a Debt_Node marking topic avoidance
4. WHEN a teach-back validation fails, THE System SHALL increase the Severity_Score for the related Debt_Node
5. THE System SHALL timestamp every Debt_Node creation and update

### Requirement 6: Calculate Severity Scores

**User Story:** As a learner, I want the system to prioritize which gaps matter most, so that I focus on high-impact learning.

#### Acceptance Criteria

1. WHEN a Debt_Node is created, THE System SHALL calculate an initial Severity_Score based on the gap type
2. WHEN a Debt_Node is updated, THE System SHALL recalculate the Severity_Score based on frequency, recency, and impact
3. THE System SHALL assign higher Severity_Scores to prerequisite concepts that block multiple other concepts
4. THE System SHALL assign higher Severity_Scores to repeatedly failed concepts
5. THE System SHALL represent Severity_Scores as integers between 0 and 100

### Requirement 7: Prompt Teach-Back Explanations

**User Story:** As a learner, I want to be asked to explain concepts back to the system, so that I validate my understanding instead of passively consuming information.

#### Acceptance Criteria

1. WHEN a Micro_Explanation is delivered, THE System SHALL prompt the learner to explain the concept in their own words
2. THE System SHALL frame teach-back prompts as teaching scenarios (e.g., "Explain this like you're teaching a junior tomorrow")
3. WHEN prompting teach-back, THE System SHALL specify what aspect of the concept to explain
4. THE System SHALL not accept "yes" or "I understand" as valid teach-back responses

### Requirement 8: Validate Teach-Back Quality

**User Story:** As a learner, I want the system to evaluate whether my explanation demonstrates true understanding, so that I know if I've actually mastered the concept.

#### Acceptance Criteria

1. WHEN a learner submits a teach-back explanation, THE Teach_Back_Validator SHALL analyze the explanation for completeness
2. WHEN analyzing teach-back, THE System SHALL detect logical holes in the learner's explanation
3. WHEN analyzing teach-back, THE System SHALL identify false confidence indicators (vague language, circular reasoning)
4. WHEN a teach-back is complete and accurate, THE System SHALL mark it as validated
5. WHEN a teach-back is incomplete or inaccurate, THE System SHALL identify specific gaps in the explanation

### Requirement 9: Update Knowledge Debt Based on Validation

**User Story:** As a learner, I want my knowledge debt to decrease only when I prove understanding, so that the system accurately reflects my true mastery.

#### Acceptance Criteria

1. WHEN a teach-back is validated as complete, THE System SHALL reduce the Severity_Score for the corresponding Debt_Node
2. WHEN a teach-back fails validation, THE System SHALL maintain or increase the Severity_Score for the corresponding Debt_Node
3. WHEN a Severity_Score reaches zero, THE System SHALL mark the Debt_Node as resolved
4. THE System SHALL not remove resolved Debt_Nodes from the graph (maintain history)

### Requirement 10: Suggest Next Learning Priority

**User Story:** As a learner, I want the system to tell me what to learn next based on my knowledge debt, so that my learning path is optimized.

#### Acceptance Criteria

1. WHEN a learning loop completes, THE System SHALL analyze the Knowledge_Debt_Graph to identify the highest priority gap
2. WHEN suggesting next priority, THE System SHALL consider Severity_Scores and prerequisite relationships
3. THE System SHALL present the next priority as a clear, actionable learning goal
4. WHEN multiple gaps have equal priority, THE System SHALL suggest the most foundational concept first

### Requirement 11: Visualize Knowledge Debt Graph

**User Story:** As a learner, I want to see a visual representation of my knowledge debt, so that I understand my learning landscape.

#### Acceptance Criteria

1. WHEN a learner requests their knowledge debt status, THE System SHALL generate a visualization of the Knowledge_Debt_Graph
2. THE System SHALL display each Debt_Node with its concept name and Severity_Score
3. THE System SHALL use visual indicators (color, size, or labels) to distinguish severity levels (High, Medium, Low, Resolved)
4. WHEN displaying the graph, THE System SHALL show prerequisite relationships between concepts

### Requirement 12: Persist Learning State

**User Story:** As a learner, I want all my interactions and knowledge debt to be saved, so that I can continue learning across multiple sessions.

#### Acceptance Criteria

1. WHEN any interaction completes, THE System SHALL persist the updated Knowledge_Debt_Graph to storage
2. WHEN a learner returns to the system, THE System SHALL load their existing Knowledge_Debt_Graph
3. THE System SHALL persist all diagnosis history, teach-back attempts, and validation results
4. WHEN storage operations fail, THE System SHALL retry and notify the learner of any data loss risk

### Requirement 13: Handle Ambiguous Input

**User Story:** As a learner, I want the system to ask clarifying questions when my input is unclear, so that diagnosis is accurate.

#### Acceptance Criteria

1. WHEN the Diagnosis_Engine cannot determine the root cause from input alone, THE System SHALL generate clarifying questions
2. THE System SHALL limit clarifying questions to a maximum of 3 per interaction
3. WHEN clarifying questions are answered, THE System SHALL incorporate the answers into the diagnosis
4. IF clarifying questions don't resolve ambiguity, THE System SHALL make a best-effort diagnosis and note the uncertainty

### Requirement 14: Detect Repeated Patterns

**User Story:** As a learner, I want the system to notice when I make the same mistakes repeatedly, so that it can address deeper misconceptions.

#### Acceptance Criteria

1. WHEN analyzing input, THE System SHALL compare current errors against historical patterns in the Knowledge_Debt_Graph
2. WHEN a pattern repeats 3 or more times, THE System SHALL flag it as a persistent misconception
3. WHEN a persistent misconception is detected, THE System SHALL adjust the diagnosis to address the underlying mental model
4. THE System SHALL track pattern frequency and time intervals between occurrences

### Requirement 15: Support Multiple Subject Domains

**User Story:** As a learner, I want to use the system for different subjects (programming, math, science), so that I have one learning platform for all topics.

#### Acceptance Criteria

1. THE System SHALL maintain separate Knowledge_Debt_Graphs for each subject domain per learner
2. WHEN a learner switches domains, THE System SHALL load the appropriate Knowledge_Debt_Graph
3. THE System SHALL support domain-specific concept taxonomies for accurate gap classification
4. WHEN creating a new domain, THE System SHALL initialize it with domain-appropriate prerequisite relationships
