# Design Document: Cognitive Gap AI Learning System

## Overview

The Cognitive Gap AI Learning System is a cognitive learning engine that implements a closed learning loop combining diagnosis, strategic memory, and mastery validation. The system architecture consists of three integrated layers:

1. **Diagnosis Layer** - Analyzes learner input to identify root causes of confusion
2. **Memory & Intelligence Layer** - Maintains a persistent Knowledge Debt Graph tracking all learning gaps
3. **Validation Layer** - Validates understanding through teach-back methodology

The system differentiates itself from traditional AI tutoring by diagnosing BEFORE explaining, maintaining strategic learning memory across all interactions, and requiring active demonstration of understanding rather than passive acknowledgment.

### Key Design Principles

- **Diagnosis-First**: Never explain without first understanding why the learner is stuck
- **Memory-Driven**: Every interaction updates a persistent knowledge model
- **Validation-Required**: Understanding is proven through teach-back, not assumed
- **Strategic Learning**: Priority is calculated based on severity, frequency, and prerequisite relationships
- **Closed Loop**: Each interaction completes a full cycle from diagnosis to validation to memory update

## Architecture

### System Components

```
┌─────────────────────────────────────────────────────────────┐
│                     Learner Interface                        │
│              (Input Collection & Visualization)              │
└────────────────────────┬────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────┐
│                   Diagnosis Engine                           │
│  ┌──────────────────────────────────────────────────────┐  │
│  │ Input Analyzer → Gap Classifier → Diagnosis Generator│  │
│  └──────────────────────────────────────────────────────┘  │
└────────────────────────┬────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────┐
│              Explanation Generator                           │
│         (Micro-Explanation Synthesis)                        │
└────────────────────────┬────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────┐
│              Teach-Back Validator                            │
│  ┌──────────────────────────────────────────────────────┐  │
│  │ Prompt Generator → Response Analyzer → Quality Scorer│  │
│  └──────────────────────────────────────────────────────┘  │
└────────────────────────┬────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────┐
│           Knowledge Debt Graph Manager                       │
│  ┌──────────────────────────────────────────────────────┐  │
│  │ Graph Store → Severity Calculator → Priority Ranker  │  │
│  └──────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
```

### Data Flow

1. **Input Phase**: Learner submits code, error, question, or "stuck" statement
2. **Diagnosis Phase**: System analyzes input and classifies cognitive gap type
3. **Explanation Phase**: System generates minimal, targeted explanation
4. **Teach-Back Phase**: System prompts learner to explain concept back
5. **Validation Phase**: System evaluates teach-back quality
6. **Update Phase**: System updates Knowledge Debt Graph based on validation
7. **Recommendation Phase**: System suggests next learning priority

### Technology Stack Considerations

- **LLM Integration**: GPT-4 or Claude for diagnosis and validation (requires prompt engineering)
- **Graph Database**: Neo4j or similar for Knowledge Debt Graph (supports relationship queries)
- **Storage**: PostgreSQL for interaction history and learner profiles
- **API Layer**: REST or GraphQL for frontend communication
- **Frontend**: React or similar for visualization and interaction

## Components and Interfaces

### 1. Input Processor

**Responsibility**: Normalize and validate learner input

**Interface**:
```typescript
interface InputProcessor {
  processInput(rawInput: string, inputType: InputType): ProcessedInput
  validateInput(input: string): ValidationResult
}

enum InputType {
  CODE_WITH_ERROR,
  NATURAL_LANGUAGE_QUESTION,
  STUCK_STATEMENT,
  MIXED
}

interface ProcessedInput {
  inputType: InputType
  content: string
  metadata: {
    timestamp: Date
    learnerId: string
    domain: string
  }
}
```

### 2. Diagnosis Engine

**Responsibility**: Analyze input and identify root cause of confusion

**Interface**:
```typescript
interface DiagnosisEngine {
  diagnose(input: ProcessedInput, knowledgeGraph: KnowledgeDebtGraph): Diagnosis
  classifyGap(input: ProcessedInput): GapType
  generateClarifyingQuestions(input: ProcessedInput): Question[]
}

enum GapType {
  CONCEPTUAL_GAP,
  MISCONCEPTION,
  MISSING_PREREQUISITE,
  COGNITIVE_OVERLOAD
}

interface Diagnosis {
  gapType: GapType
  rootCause: string
  affectedConcepts: string[]
  confidence: number // 0-1
  clarificationNeeded: boolean
}
```

### 3. Explanation Generator

**Responsibility**: Create targeted micro-explanations

**Interface**:
```typescript
interface ExplanationGenerator {
  generateMicroExplanation(diagnosis: Diagnosis, knowledgeGraph: KnowledgeDebtGraph): MicroExplanation
  filterKnownConcepts(concepts: string[], knowledgeGraph: KnowledgeDebtGraph): string[]
}

interface MicroExplanation {
  content: string
  targetConcept: string
  prerequisites: string[]
  estimatedReadingTime: number // seconds
}
```

### 4. Teach-Back Validator

**Responsibility**: Prompt for and evaluate teach-back explanations

**Interface**:
```typescript
interface TeachBackValidator {
  generatePrompt(concept: string, diagnosis: Diagnosis): TeachBackPrompt
  validateExplanation(learnerExplanation: string, targetConcept: string): ValidationResult
  detectLogicalHoles(explanation: string): LogicalHole[]
  detectFalseConfidence(explanation: string): ConfidenceIndicator[]
}

interface TeachBackPrompt {
  promptText: string
  targetConcept: string
  evaluationCriteria: string[]
}

interface ValidationResult {
  isValid: boolean
  completenessScore: number // 0-100
  accuracyScore: number // 0-100
  gaps: string[]
  feedback: string
}

interface LogicalHole {
  description: string
  severity: 'minor' | 'major' | 'critical'
}

interface ConfidenceIndicator {
  type: 'vague_language' | 'circular_reasoning' | 'hedging'
  example: string
}
```

### 5. Knowledge Debt Graph Manager

**Responsibility**: Maintain and query the knowledge debt graph

**Interface**:
```typescript
interface KnowledgeDebtGraphManager {
  createGraph(learnerId: string, domain: string): KnowledgeDebtGraph
  loadGraph(learnerId: string, domain: string): KnowledgeDebtGraph
  addDebtNode(graph: KnowledgeDebtGraph, node: DebtNode): void
  updateDebtNode(graph: KnowledgeDebtGraph, nodeId: string, update: DebtNodeUpdate): void
  calculateSeverity(node: DebtNode, graph: KnowledgeDebtGraph): number
  getHighestPriorityGap(graph: KnowledgeDebtGraph): DebtNode | null
  detectPatterns(graph: KnowledgeDebtGraph): Pattern[]
  persist(graph: KnowledgeDebtGraph): Promise<void>
}

interface KnowledgeDebtGraph {
  learnerId: string
  domain: string
  nodes: Map<string, DebtNode>
  edges: Map<string, PrerequisiteRelationship>
  createdAt: Date
  lastUpdated: Date
}

interface DebtNode {
  id: string
  concept: string
  gapType: GapType
  severityScore: number // 0-100
  occurrenceCount: number
  firstSeen: Date
  lastSeen: Date
  teachBackAttempts: number
  teachBackSuccesses: number
  resolved: boolean
}

interface PrerequisiteRelationship {
  fromConcept: string
  toConcept: string
  strength: number // 0-1, how critical the prerequisite is
}

interface Pattern {
  type: 'repeated_error' | 'topic_avoidance' | 'persistent_misconception'
  concepts: string[]
  frequency: number
  firstDetected: Date
}
```

### 6. Priority Recommender

**Responsibility**: Suggest next learning priority

**Interface**:
```typescript
interface PriorityRecommender {
  recommendNext(graph: KnowledgeDebtGraph): Recommendation
  rankGaps(gaps: DebtNode[], graph: KnowledgeDebtGraph): DebtNode[]
}

interface Recommendation {
  concept: string
  reason: string
  estimatedImpact: number // 0-100
  prerequisitesNeeded: string[]
}
```

### 7. Visualization Generator

**Responsibility**: Create visual representations of knowledge debt

**Interface**:
```typescript
interface VisualizationGenerator {
  generateGraphVisualization(graph: KnowledgeDebtGraph): GraphVisualization
  categorizeByseverity(nodes: DebtNode[]): SeverityCategories
}

interface GraphVisualization {
  nodes: VisualNode[]
  edges: VisualEdge[]
  layout: 'hierarchical' | 'force-directed' | 'circular'
}

interface VisualNode {
  id: string
  label: string
  severity: 'high' | 'medium' | 'low' | 'resolved'
  color: string
  size: number
}

interface SeverityCategories {
  high: DebtNode[]    // severity 70-100
  medium: DebtNode[]  // severity 30-69
  low: DebtNode[]     // severity 1-29
  resolved: DebtNode[] // severity 0
}
```

## Data Models

### Core Data Structures

**Learner Profile**:
```typescript
interface LearnerProfile {
  id: string
  name: string
  email: string
  domains: string[] // e.g., ['programming', 'mathematics']
  createdAt: Date
  lastActive: Date
}
```

**Interaction Record**:
```typescript
interface InteractionRecord {
  id: string
  learnerId: string
  domain: string
  timestamp: Date
  input: ProcessedInput
  diagnosis: Diagnosis
  explanation: MicroExplanation
  teachBackPrompt: TeachBackPrompt
  teachBackResponse: string | null
  validationResult: ValidationResult | null
  graphUpdateSummary: string
}
```

**Severity Calculation Formula**:
```
severity = (
  baseWeight[gapType] * 40 +
  occurrenceCount * 10 +
  recencyFactor * 20 +
  prerequisiteImpact * 20 +
  failureRate * 10
)

where:
  baseWeight = {
    MISSING_PREREQUISITE: 1.0,
    MISCONCEPTION: 0.9,
    CONCEPTUAL_GAP: 0.8,
    COGNITIVE_OVERLOAD: 0.6
  }
  
  recencyFactor = 1.0 if seen in last 24h, decays exponentially
  
  prerequisiteImpact = count of concepts blocked by this gap / total concepts
  
  failureRate = teachBackAttempts > 0 ? 
    (1 - teachBackSuccesses / teachBackAttempts) : 0.5
```

### Storage Schema

**PostgreSQL Tables**:
- `learners`: Profile information
- `interactions`: Full interaction history
- `teach_back_attempts`: Detailed teach-back records

**Graph Database (Neo4j)**:
- Node type: `Concept` (properties: name, domain)
- Node type: `DebtNode` (properties: all DebtNode fields)
- Relationship: `PREREQUISITE_OF` (properties: strength)
- Relationship: `HAS_DEBT` (connects Learner to DebtNode)

## Correctness Properties

*A property is a characteristic or behavior that should hold true across all valid executions of a system—essentially, a formal statement about what the system should do. Properties serve as the bridge between human-readable specifications and machine-verifiable correctness guarantees.*

