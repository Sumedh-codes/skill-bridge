# Design Document: SkillBridge

## 1️⃣ Problem Statement

India has over 450 million informal workers—plumbers, electricians, construction workers, cooks—with valuable real-world skills but no formal credentials or resumes. They struggle to access better-paying formal jobs because they can't articulate their skills in employer-friendly terms, don't know which skills are in demand, and lack access to job portals. Meanwhile, employers struggle to find skilled workers because traditional hiring methods don't reach this demographic. This creates a massive skills-to-jobs gap, trapping millions in low-wage informal work despite having marketable abilities. SkillBridge solves this by meeting workers where they are—on WhatsApp, in their language, using their voice.

## 2️⃣ Proposed Solution

**What is SkillBridge?**
SkillBridge is a voice-first AI system accessible via WhatsApp that conducts conversational interviews with informal workers in their local language (Hindi, Tamil, Telugu, Bengali, Marathi). Workers simply speak about their experience—no forms, no typing, no resume writing.

**How it solves the problem:**
- **Voice-to-Profile**: AI extracts structured skill profiles from natural conversation, translating informal terminology ("I fix water pipes") into formal skills ("Residential Plumbing - 5 years")
- **Market Intelligence**: Analyzes real-time job market data to identify high-demand skills in the worker's location
- **Smart Recommendations**: Calculates ROI for skill upgrades and recommends the single best skill to learn for maximum salary increase
- **Job Matching**: Connects workers with relevant job opportunities based on their skills and location

**What makes it different:**
- **Zero literacy barrier**: Voice-only interface, no reading/writing required
- **Hyper-local**: Market analysis specific to worker's city/state
- **ROI-driven**: Recommends skills based on actual salary impact, not generic advice
- **Accessible**: Works on any phone with WhatsApp, no app installation needed

## 3️⃣ System Architecture

### High-Level Architecture Diagram

```
Worker (WhatsApp) 
    ↓
WhatsApp Business API
    ↓
API Gateway / Webhook Handler
    ↓
┌─────────────────────────────────────────────────┐
│         Conversation Manager (Orchestrator)      │
└─────────────────────────────────────────────────┘
    ↓                    ↓                    ↓
Voice Processor    Skill Extractor    Job Matcher
    ↓                    ↓                    ↓
STT/TTS APIs      NLP/LLM (GPT-4)   Market Data
    ↓                    ↓                    ↓
┌─────────────────────────────────────────────────┐
│              PostgreSQL Database                 │
│  (Users, Profiles, Skills, Jobs, Sessions)      │
└─────────────────────────────────────────────────┘
```

### Component Breakdown

**Frontend (User Interface)**:
- **WhatsApp** - The only interface workers interact with
- Supports voice messages, text messages, and media
- No custom app required, works on any phone

**Backend (Server Logic)**:
- **API Gateway**: Receives WhatsApp webhooks, handles message routing
- **Conversation Manager**: Orchestrates interview flow, maintains session state
- **Voice Processor**: Transcribes audio to text, synthesizes text to speech
- **Skill Extractor**: Uses NLP to extract skills from conversation
- **Skill Profiler**: Creates structured skill profiles with validation
- **Skill Signal Engine**: Analyzes market demand, calculates ROI, generates recommendations
- **Job Matcher**: Finds relevant job opportunities based on skills and location

**Database**:
- **PostgreSQL** for structured data (users, profiles, skills, jobs, sessions)
- **Redis** for session caching and message queuing
- **pgvector** extension for semantic skill matching using embeddings

**AI Models**:
- **Speech-to-Text**: Google Cloud Speech-to-Text or OpenAI Whisper (multi-language support)
- **Text-to-Speech**: Google Cloud TTS or Azure TTS
- **NLP/LLM**: GPT-4 or Claude for conversation management and skill extraction
- **Embeddings**: OpenAI text-embedding-ada-002 for semantic skill matching

**External Services**:
- **WhatsApp Business API** (Cloud API or On-Premises)
- **Google Cloud Speech APIs** or **Azure Speech Services**
- **OpenAI API** for LLM and embeddings
- **Job Aggregator APIs** (Naukri, Indeed, LinkedIn)

**Architecture Pattern**:
The system follows a microservices pattern with clear separation between:
- **Communication Layer**: WhatsApp integration and message handling
- **Voice Processing Layer**: Speech-to-text, NLP, and conversation management
- **Skill Intelligence Layer**: Skill extraction, profiling, and market analysis
- **Job Matching Layer**: Opportunity identification and recommendation
- **Data Layer**: Persistent storage and retrieval

## 4️⃣ Tech Stack

### Frontend
- **WhatsApp** - Primary user interface (no custom frontend needed)
- **WhatsApp Business API** - Message handling and webhook integration

### Backend
- **FastAPI** or **Node.js (Express)** - REST API server
- **Python** - Core business logic, AI/ML integration
- **Redis** - Session management, message queuing, caching

### Database
- **PostgreSQL** - Primary database for structured data
- **pgvector** - Vector extension for semantic skill matching
- **Redis** - In-memory cache and session store

### AI/ML
- **OpenAI GPT-4** or **Claude** - Conversation management, skill extraction
- **Google Cloud Speech-to-Text** or **OpenAI Whisper** - Voice transcription
- **Google Cloud TTS** or **Azure TTS** - Text-to-speech synthesis
- **OpenAI Embeddings** (text-embedding-ada-002) - Semantic skill matching

### External Services
- **WhatsApp Business API** - Message delivery and webhooks
- **Google Cloud Speech APIs** - STT/TTS processing
- **OpenAI API** - LLM and embeddings
- **Job Portal APIs** - Naukri, Indeed, LinkedIn (for job data)

### Hosting & Deployment
- **Backend**: AWS EC2, Google Cloud Run, or Render
- **Database**: AWS RDS PostgreSQL or managed PostgreSQL
- **Redis**: AWS ElastiCache or Redis Cloud
- **Storage**: AWS S3 or Google Cloud Storage (for audio archives)

### Why This Stack?
- **Realistic**: All services have free tiers or trial credits for hackathon
- **Buildable in 24-48 hours**: Proven APIs, minimal custom ML training needed
- **Scalable**: Can handle 1000+ concurrent users with proper configuration
- **Cost-effective**: WhatsApp Business API has free sandbox, OpenAI has affordable pricing

## 5️⃣ User Flow

### Complete User Journey

**Step 1: Onboarding**
1. Worker sends "Hi" to SkillBridge WhatsApp number
2. System responds: "Welcome! Please select your language: 1-Hindi, 2-Tamil, 3-Telugu, 4-Bengali, 5-Marathi"
3. Worker selects language (voice or text)

**Step 2: Voice Interview**
4. System asks: "Tell me about your work. What do you do?" (in selected language)
5. Worker sends voice message describing their skills and experience
6. System transcribes audio and extracts skills
7. System asks follow-up questions to clarify:
   - "How many years have you been doing this?"
   - "Can you describe a recent project?"
   - "What tools or equipment do you use?"
8. Conversation continues until profile is complete (5-10 exchanges)

**Step 3: Profile Creation**
9. System generates structured skill profile
10. System sends summary: "I've created your profile with these skills: [list]. Is this correct?"
11. Worker confirms or requests changes

**Step 4: Market Analysis**
12. System analyzes job market for worker's location
13. System identifies skill gaps and calculates ROI

**Step 5: Recommendations**
14. System sends: "Based on your skills and local job market, learning [SKILL] could increase your monthly income by ₹[AMOUNT]"
15. System provides learning resources and estimated time to learn

**Step 6: Job Matching**
16. System finds relevant job opportunities
17. System sends: "I found [N] jobs matching your skills in [LOCATION]"
18. Worker can browse jobs, see requirements, and get application info

**Step 7: Follow-up**
19. Worker can return anytime to update profile, get new recommendations, or see new jobs
20. System maintains conversation history and context

### Quick Flow Diagram
```
User → Language Selection → Voice Interview → Skill Extraction
  ↓
Profile Creation → Market Analysis → Skill Gap Identification
  ↓
ROI Calculation → Recommendations → Job Matching → Application
```

## 6️⃣ Features

### Core Features (Must Have - MVP)

1. **Voice-Based Skill Assessment**
   - WhatsApp voice message input
   - Multi-language speech-to-text (Hindi, Tamil, Telugu, Bengali, Marathi)
   - Conversational interview flow with follow-up questions
   - Informal terminology mapping to formal skills

2. **Structured Skill Profiling**
   - Extract skills, proficiency levels, and experience from conversation
   - Validate skills against stated experience
   - Generate structured skill profiles
   - Persist profiles to database

3. **Market-Aligned Skill Analysis**
   - Compare worker skills against job market demand
   - Identify skill gaps based on local market data
   - Rank gaps by potential salary increase

4. **Skill Upgrade Recommendations**
   - Calculate ROI for skill upgrades
   - Recommend top skill to learn for maximum income increase
   - Provide estimated salary increase and learning time

5. **Job Opportunity Matching**
   - Match profiles with relevant jobs based on skills and location
   - Display job details (title, employer, salary, requirements)
   - Highlight skill alignment and gaps

6. **WhatsApp Integration**
   - Webhook handling for incoming messages
   - Message queuing and retry logic
   - Support for voice and text responses

### Bonus Features (If Time Permits)

7. **Text-to-Speech Responses**
   - Voice responses in addition to text
   - Accessibility for low-literacy users

8. **Skill Verification**
   - Mark skills as self-reported vs. verified
   - Integration with third-party verification services

9. **Learning Resource Recommendations**
   - Suggest courses, certifications, training centers
   - Include cost, duration, and location

10. **Near-Match Job Suggestions**
    - Show jobs requiring 1-2 additional skills
    - Provide learning paths to qualify

11. **Data Privacy Dashboard**
    - Allow workers to view, update, or delete their data
    - Consent management for data sharing

12. **Analytics Dashboard**
    - Track conversation completion rates
    - Monitor skill extraction accuracy
    - Measure job match success rates

## 7️⃣ Database Design

### Database Schema Overview

The database uses PostgreSQL with the following core tables:

### **Users Table**
Stores basic worker information and preferences.

| Field | Type | Description |
|-------|------|-------------|
| user_id | UUID | Primary key |
| phone_number | VARCHAR(20) | WhatsApp number (unique) |
| preferred_language | VARCHAR(5) | hi, ta, te, bn, mr, en |
| location_city | VARCHAR(100) | Worker's city |
| location_state | VARCHAR(100) | Worker's state |
| location_pincode | VARCHAR(10) | Optional pincode |
| created_at | TIMESTAMP | Account creation time |
| updated_at | TIMESTAMP | Last profile update |
| last_active | TIMESTAMP | Last interaction time |

### **Skill Profiles Table**
Stores aggregated skill profile information.

| Field | Type | Description |
|-------|------|-------------|
| profile_id | UUID | Primary key |
| user_id | UUID | Foreign key to users |
| primary_occupation | VARCHAR(100) | Main job title |
| total_years_experience | INTEGER | Total work experience |
| completeness_score | DECIMAL(5,2) | Profile completeness (0-100) |
| verification_status | VARCHAR(20) | UNVERIFIED, PARTIALLY_VERIFIED, VERIFIED |
| created_at | TIMESTAMP | Profile creation time |
| updated_at | TIMESTAMP | Last update time |

### **Skills Table**
Stores individual skills for each profile.

| Field | Type | Description |
|-------|------|-------------|
| skill_id | UUID | Primary key |
| profile_id | UUID | Foreign key to skill_profiles |
| skill_name | VARCHAR(200) | Skill name (e.g., "Residential Plumbing") |
| category | VARCHAR(50) | ELECTRICAL, PLUMBING, CONSTRUCTION, etc. |
| proficiency_level | VARCHAR(20) | BEGINNER, INTERMEDIATE, ADVANCED, EXPERT |
| years_of_experience | INTEGER | Years working with this skill |
| verification_status | VARCHAR(20) | SELF_REPORTED, VERIFIED, CERTIFIED |
| created_at | TIMESTAMP | Skill added time |

### **Conversation Sessions Table**
Tracks active and completed conversations.

| Field | Type | Description |
|-------|------|-------------|
| session_id | UUID | Primary key |
| user_id | UUID | Foreign key to users |
| state | VARCHAR(50) | Current conversation state |
| language | VARCHAR(5) | Session language |
| created_at | TIMESTAMP | Session start time |
| last_activity | TIMESTAMP | Last message time |
| completed_at | TIMESTAMP | Session completion time (nullable) |

### **Conversation Messages Table**
Stores all messages in a conversation.

| Field | Type | Description |
|-------|------|-------------|
| message_id | UUID | Primary key |
| session_id | UUID | Foreign key to conversation_sessions |
| direction | VARCHAR(10) | INBOUND or OUTBOUND |
| message_type | VARCHAR(10) | TEXT or AUDIO |
| content | TEXT | Message text content |
| audio_url | VARCHAR(500) | URL to audio file (if audio) |
| transcription | TEXT | Transcribed text (if audio) |
| created_at | TIMESTAMP | Message timestamp |

### **Market Demand Table**
Stores job market demand data for skills.

| Field | Type | Description |
|-------|------|-------------|
| demand_id | UUID | Primary key |
| skill_name | VARCHAR(200) | Skill name |
| location_city | VARCHAR(100) | City for demand data |
| location_state | VARCHAR(100) | State for demand data |
| demand_score | DECIMAL(5,2) | Demand score (0-100) |
| job_posting_count | INTEGER | Number of job postings |
| average_salary | DECIMAL(10,2) | Average salary for skill |
| growth_rate | DECIMAL(5,2) | Year-over-year growth % |
| data_date | DATE | Date of data collection |
| created_at | TIMESTAMP | Record creation time |

### **Job Listings Table**
Stores available job opportunities.

| Field | Type | Description |
|-------|------|-------------|
| job_id | UUID | Primary key |
| title | VARCHAR(200) | Job title |
| employer | VARCHAR(200) | Company name |
| description | TEXT | Job description |
| location_city | VARCHAR(100) | Job location city |
| location_state | VARCHAR(100) | Job location state |
| salary_min | DECIMAL(10,2) | Minimum salary |
| salary_max | DECIMAL(10,2) | Maximum salary |
| employment_type | VARCHAR(20) | FULL_TIME, PART_TIME, CONTRACT, TEMPORARY |
| posted_date | DATE | Job posting date |
| expiry_date | DATE | Application deadline |
| source | VARCHAR(100) | Job source (Naukri, Indeed, etc.) |
| source_url | VARCHAR(500) | Original job posting URL |
| created_at | TIMESTAMP | Record creation time |

### **Job Skills Table** (Many-to-Many)
Links jobs to required skills.

| Field | Type | Description |
|-------|------|-------------|
| job_id | UUID | Foreign key to job_listings |
| skill_name | VARCHAR(200) | Required skill name |
| is_required | BOOLEAN | True if required, false if preferred |

**Primary Key**: (job_id, skill_name)

### **Skill Embeddings Table** (For Semantic Matching)
Stores vector embeddings for semantic skill matching.

| Field | Type | Description |
|-------|------|-------------|
| skill_name | VARCHAR(200) | Primary key |
| embedding | vector(1536) | Vector embedding (pgvector) |
| created_at | TIMESTAMP | Embedding creation time |

### Relationships

```
users (1) ──→ (many) skill_profiles
skill_profiles (1) ──→ (many) skills
users (1) ──→ (many) conversation_sessions
conversation_sessions (1) ──→ (many) conversation_messages
job_listings (many) ←──→ (many) skills [via job_skills]
```

### Indexes

- `idx_users_phone` on users(phone_number)
- `idx_profiles_user` on skill_profiles(user_id)
- `idx_skills_profile` on skills(profile_id)
- `idx_skills_category` on skills(category)
- `idx_sessions_user` on conversation_sessions(user_id)
- `idx_sessions_state` on conversation_sessions(state)
- `idx_messages_session` on conversation_messages(session_id)
- `idx_demand_skill_location` on market_demand(skill_name, location_city, data_date)
- `idx_jobs_location` on job_listings(location_city, location_state)
- `idx_jobs_posted` on job_listings(posted_date)
- `idx_job_skills_skill` on job_skills(skill_name)
- `idx_skill_embeddings_vector` on skill_embeddings using ivfflat (embedding vector_cosine_ops)

## 8️⃣ Future Scope

### Post-Hackathon Improvements

**1. Enhanced AI Capabilities**
- Fine-tune LLM on Indian informal sector terminology for better skill extraction
- Add computer vision for skill verification (e.g., photos of completed work)
- Implement sentiment analysis to detect worker confidence levels
- Add voice emotion detection to improve conversation flow

**2. Expanded Language Support**
- Add Kannada, Malayalam, Gujarati, Punjabi, Odia
- Support code-switching (mixing languages in one conversation)
- Regional dialect recognition

**3. Skill Verification Ecosystem**
- Partner with training centers for skill certification
- Integrate with government skill development programs (NSDC, PMKVY)
- Employer-verified skill endorsements
- Peer verification from other workers

**4. Advanced Job Matching**
- Real-time job alerts via WhatsApp
- Application tracking and status updates
- Interview preparation assistance
- Salary negotiation guidance
- Direct application submission through platform

**5. Learning Marketplace**
- Curated skill development courses
- Micro-learning modules via WhatsApp
- Video tutorials in local languages
- Gamified learning with progress tracking
- Connect workers with mentors

**6. Monetization Strategy**
- **Freemium Model**: Basic profile and job matching free, premium features paid
- **Employer Subscriptions**: Charge employers for access to verified worker profiles
- **Training Partner Commissions**: Revenue share with course providers
- **Government Partnerships**: Subsidized access through skill development programs
- **Job Placement Fees**: Commission on successful placements (paid by employers)

**7. Scaling Strategy**
- **Geographic Expansion**: Start with 3-4 major cities, expand to tier-2/3 cities
- **Vertical Expansion**: Add more skill categories (beauty, retail, hospitality, healthcare)
- **Horizontal Expansion**: Expand to other countries with large informal sectors (Bangladesh, Indonesia, Philippines)
- **Infrastructure**: Move to Kubernetes for auto-scaling, implement CDN for audio files
- **Database**: Implement read replicas, sharding by geographic region

**8. Social Impact Features**
- Worker community forums for peer support
- Success story sharing to inspire others
- Financial literacy modules
- Rights awareness and labor law education
- Emergency job assistance during crises

**9. Analytics & Insights**
- Employer dashboard showing skill supply trends
- Government dashboard for policy insights
- Worker dashboard showing career progression
- Market intelligence reports for training providers

**10. Platform Integrations**
- Integration with payment platforms for salary transparency
- Integration with background verification services
- Integration with government ID systems (Aadhaar) for verification
- Integration with gig economy platforms (Uber, Swiggy, Urban Company)

### Success Metrics (6-12 Months)
- **User Acquisition**: 100,000+ worker profiles created
- **Job Placements**: 10,000+ successful job matches
- **Skill Upgrades**: 25,000+ workers complete recommended training
- **Income Impact**: Average 30% salary increase for placed workers
- **Employer Adoption**: 1,000+ employers using platform for hiring
- **Geographic Coverage**: 20+ cities across India

---

## Technical Implementation Details

### 1. WhatsApp Gateway

**Responsibility**: Handle all communication with WhatsApp Business API

**Interface**:
```typescript
interface WhatsAppGateway {
  // Receive incoming messages from WhatsApp
  receiveMessage(webhookPayload: WebhookPayload): Promise<void>
  
  // Send text message to worker
  sendTextMessage(phoneNumber: string, message: string): Promise<MessageStatus>
  
  // Send voice message to worker
  sendVoiceMessage(phoneNumber: string, audioUrl: string): Promise<MessageStatus>
  
  // Download voice message from WhatsApp
  downloadVoiceMessage(mediaId: string): Promise<AudioBuffer>
  
  // Mark message as read
  markAsRead(messageId: string): Promise<void>
}

interface WebhookPayload {
  messageId: string
  from: string  // Phone number
  timestamp: number
  type: 'text' | 'audio' | 'image'
  content: string | MediaReference
}

interface MessageStatus {
  messageId: string
  status: 'sent' | 'delivered' | 'read' | 'failed'
  timestamp: number
}
```

### 2. Conversation Manager

**Responsibility**: Orchestrate conversation flow, maintain context, and coordinate between components

**Interface**:
```typescript
interface ConversationManager {
  // Handle incoming message and route to appropriate handler
  handleMessage(phoneNumber: string, message: IncomingMessage): Promise<void>
  
  // Get or create conversation session
  getSession(phoneNumber: string): Promise<ConversationSession>
  
  // Update conversation state
  updateState(sessionId: string, state: ConversationState): Promise<void>
  
  // Generate next question based on current context
  generateNextQuestion(session: ConversationSession): Promise<string>
  
  // Determine if interview is complete
  isInterviewComplete(session: ConversationSession): boolean
}

interface ConversationSession {
  sessionId: string
  phoneNumber: string
  language: Language
  state: ConversationState
  extractedSkills: Skill[]
  conversationHistory: Message[]
  createdAt: Date
  lastActivity: Date
}

type ConversationState = 
  | 'LANGUAGE_SELECTION'
  | 'INITIAL_GREETING'
  | 'SKILL_DISCOVERY'
  | 'SKILL_DEPTH_PROBING'
  | 'EXPERIENCE_VALIDATION'
  | 'PROFILE_REVIEW'
  | 'MARKET_ANALYSIS'
  | 'RECOMMENDATION'
  | 'JOB_MATCHING'
  | 'COMPLETED'

type Language = 'hi' | 'ta' | 'te' | 'bn' | 'mr' | 'en'
```

### 3. Voice Processor

**Responsibility**: Convert speech to text and text to speech, handle audio processing

**Interface**:
```typescript
interface VoiceProcessor {
  // Transcribe audio to text
  transcribe(audio: AudioBuffer, language: Language): Promise<TranscriptionResult>
  
  // Synthesize text to speech
  synthesize(text: string, language: Language): Promise<AudioBuffer>
  
  // Detect language from audio
  detectLanguage(audio: AudioBuffer): Promise<Language>
}

interface TranscriptionResult {
  text: string
  confidence: number
  language: Language
  duration: number
}
```

### 4. Skill Extractor

**Responsibility**: Extract structured skills from conversational text using NLP

**Interface**:
```typescript
interface SkillExtractor {
  // Extract skills from conversation text
  extractSkills(text: string, context: ConversationContext): Promise<ExtractedSkill[]>
  
  // Validate skill claim against experience
  validateSkill(skill: ExtractedSkill, experience: string): Promise<ValidationResult>
  
  // Map informal terminology to formal skill categories
  normalizeSkill(informalSkill: string): Promise<FormalSkill>
  
  // Generate follow-up questions for skill depth assessment
  generateProbe(skill: ExtractedSkill): Promise<string>
}

interface ExtractedSkill {
  skillName: string
  category: SkillCategory
  informalTerms: string[]
  confidence: number
  mentionedInContext: string
}

interface FormalSkill {
  standardName: string
  category: SkillCategory
  relatedSkills: string[]
  industryTerms: string[]
}

type SkillCategory = 
  | 'ELECTRICAL'
  | 'PLUMBING'
  | 'CONSTRUCTION'
  | 'MECHANICAL'
  | 'COOKING'
  | 'CARPENTRY'
  | 'WELDING'
  | 'PAINTING'
  | 'MASONRY'
  | 'HVAC'
  | 'OTHER'
```

### 5. Skill Profiler

**Responsibility**: Create and manage structured skill profiles

**Interface**:
```typescript
interface SkillProfiler {
  // Create skill profile from extracted skills
  createProfile(phoneNumber: string, skills: ExtractedSkill[], experience: ExperienceData): Promise<SkillProfile>
  
  // Update existing profile
  updateProfile(profileId: string, updates: Partial<SkillProfile>): Promise<SkillProfile>
  
  // Get profile by phone number
  getProfile(phoneNumber: string): Promise<SkillProfile | null>
  
  // Calculate profile completeness score
  calculateCompleteness(profile: SkillProfile): number
}

interface SkillProfile {
  profileId: string
  phoneNumber: string
  language: Language
  skills: ProfiledSkill[]
  experience: ExperienceData
  location: Location
  completenessScore: number
  verificationStatus: VerificationStatus
  createdAt: Date
  updatedAt: Date
}

interface ProfiledSkill {
  skillId: string
  skillName: string
  category: SkillCategory
  proficiencyLevel: 'BEGINNER' | 'INTERMEDIATE' | 'ADVANCED' | 'EXPERT'
  yearsOfExperience: number
  projectExamples: string[]
  verificationStatus: 'SELF_REPORTED' | 'VERIFIED' | 'CERTIFIED'
}

interface ExperienceData {
  totalYearsWorking: number
  primaryOccupation: string
  workLocations: string[]
  notableProjects: string[]
}

interface Location {
  city: string
  state: string
  pincode?: string
}

type VerificationStatus = 'UNVERIFIED' | 'PARTIALLY_VERIFIED' | 'VERIFIED'
```

### 6. Skill Signal Engine

**Responsibility**: Analyze job market data, identify skill gaps, and calculate ROI for skill upgrades

**Interface**:
```typescript
interface SkillSignalEngine {
  // Analyze market demand for skills in a location
  analyzeMarketDemand(location: Location): Promise<MarketDemandReport>
  
  // Identify skill gaps for a profile
  identifySkillGaps(profile: SkillProfile, marketData: MarketDemandReport): Promise<SkillGap[]>
  
  // Calculate ROI for acquiring a skill
  calculateROI(currentProfile: SkillProfile, targetSkill: string): Promise<ROIAnalysis>
  
  // Get top skill recommendation
  getTopRecommendation(profile: SkillProfile): Promise<SkillRecommendation>
  
  // Update market data cache
  refreshMarketData(): Promise<void>
}

interface MarketDemandReport {
  location: Location
  topDemandSkills: DemandSkill[]
  salaryRanges: Map<string, SalaryRange>
  hiringTrends: TrendData[]
  generatedAt: Date
}

interface DemandSkill {
  skillName: string
  demandScore: number  // 0-100
  jobPostingCount: number
  averageSalary: number
  growthRate: number  // Percentage
}

interface SkillGap {
  missingSkill: string
  category: SkillCategory
  demandScore: number
  estimatedSalaryIncrease: number
  learningTimeMonths: number
  prerequisites: string[]
  roiScore: number
}

interface ROIAnalysis {
  targetSkill: string
  currentEstimatedSalary: number
  projectedSalary: number
  salaryIncrease: number
  learningTimeMonths: number
  roiScore: number
  marketDemand: number
  confidence: number
}

interface SkillRecommendation {
  recommendedSkill: string
  rationale: string
  estimatedMonthlySalaryIncrease: number
  learningPath: string[]
  resources: LearningResource[]
}

interface LearningResource {
  title: string
  type: 'COURSE' | 'CERTIFICATION' | 'APPRENTICESHIP' | 'TRAINING_CENTER'
  provider: string
  duration: string
  cost: number
  location?: string
  url?: string
}

interface SalaryRange {
  min: number
  max: number
  median: number
  currency: string
}
```

### 7. Job Matcher

**Responsibility**: Match worker profiles with relevant job opportunities

**Interface**:
```typescript
interface JobMatcher {
  // Find job matches for a profile
  findMatches(profile: SkillProfile, limit: number): Promise<JobMatch[]>
  
  // Calculate match score between profile and job
  calculateMatchScore(profile: SkillProfile, job: JobListing): Promise<MatchScore>
  
  // Get job details
  getJobDetails(jobId: string): Promise<JobListing>
  
  // Find near-matches (jobs with small skill gaps)
  findNearMatches(profile: SkillProfile): Promise<NearMatch[]>
}

interface JobMatch {
  jobId: string
  jobTitle: string
  employer: string
  location: Location
  salaryRange: SalaryRange
  requiredSkills: string[]
  matchScore: number
  matchedSkills: string[]
  distance: number  // km from worker
  postedDate: Date
  applicationUrl?: string
  contactInfo?: ContactInfo
}

interface MatchScore {
  overallScore: number  // 0-100
  skillMatchPercentage: number
  locationScore: number
  salaryAlignment: number
  experienceMatch: number
}

interface NearMatch {
  job: JobMatch
  missingSkills: string[]
  skillGapAnalysis: string
}

interface JobListing {
  jobId: string
  title: string
  employer: string
  description: string
  requiredSkills: string[]
  preferredSkills: string[]
  location: Location
  salaryRange: SalaryRange
  employmentType: 'FULL_TIME' | 'PART_TIME' | 'CONTRACT' | 'TEMPORARY'
  experienceRequired: string
  postedDate: Date
  expiryDate: Date
  source: string
}

interface ContactInfo {
  name?: string
  phone?: string
  email?: string
  whatsapp?: string
}
```

---

## Correctness Properties

*A property is a characteristic or behavior that should hold true across all valid executions of a system—essentially, a formal statement about what the system should do. Properties serve as the bridge between human-readable specifications and machine-verifiable correctness guarantees.*


### Property 1: Language Consistency Throughout Interview

*For any* worker session with a selected language, all system-generated messages and responses throughout the interview should be in that selected language, and if the worker switches languages mid-conversation, all subsequent messages should adapt to the new language.

**Validates: Requirements 1.1, 7.2, 7.5**

### Property 2: Ambiguous Response Follow-up

*For any* worker response that is ambiguous or incomplete (lacks specific details about skills, experience duration, or project examples), the system should generate follow-up questions to clarify the information before proceeding.

**Validates: Requirements 1.3**

### Property 3: Informal Terminology Mapping

*For any* informal terminology or local job title mentioned by a worker, the system should map it to a valid formal skill category from the defined SkillCategory enumeration.

**Validates: Requirements 1.4**

### Property 4: Interview Completion Generates Profile

*For any* completed voice interview, the system should generate a structured Skill_Profile containing at least one skill with all required fields (skillName, category, proficiencyLevel, yearsOfExperience).

**Validates: Requirements 1.5**

### Property 5: Skill Extraction Specificity

*For any* conversation text containing skill mentions, the extracted skills should be specific technical skills, tools, or competencies (not vague terms like "good worker" or "hard worker"), and each should be assigned to a valid skill category.

**Validates: Requirements 2.1**

### Property 6: Data Structure Completeness

*For any* generated data structure (Skill_Profile or Job_Match), all required fields as defined in the interface should be present and non-null, ensuring complete information for downstream processing.

**Validates: Requirements 2.2, 5.2**

### Property 7: Project Experience Extraction

*For any* conversation where a worker mentions project experience or specific work examples, those examples should be extracted and included in the Skill_Profile's projectExamples field for the relevant skill.

**Validates: Requirements 2.3**

### Property 8: Skill-Experience Validation

*For any* skill added to a Skill_Profile, there should be supporting evidence in the stated experience (either years of experience > 0 or at least one project example), otherwise the skill should be flagged for clarification or excluded.

**Validates: Requirements 2.4, 9.1**

### Property 9: Profile Persistence Round-Trip

*For any* Skill_Profile created by the system, immediately querying storage with the profile's identifier should return an equivalent profile with all the same skills and experience data.

**Validates: Requirements 2.5**

### Property 10: Market Comparison Execution

*For any* Skill_Profile analyzed by the Skill_Signal_Engine, the analysis should produce market comparison data that includes demand scores for the worker's existing skills in their geographic location.

**Validates: Requirements 3.1**

### Property 11: Geographic Skill Gap Prioritization

*For any* list of identified skill gaps, the gaps should be ordered such that skills with higher local market demand scores appear before skills with lower local demand scores in the same location.

**Validates: Requirements 3.2**

### Property 12: Demand Calculation Factors

*For any* skill demand score calculation, the calculation should incorporate job posting frequency, salary data, and hiring trend data, with changes to any of these factors resulting in a different demand score.

**Validates: Requirements 3.3**

### Property 13: Skill Gap Salary Ranking

*For any* presentation of skill gaps to a worker, the gaps should be ranked in descending order by potential salary increase, with the highest salary increase appearing first.

**Validates: Requirements 3.4**

### Property 14: Top ROI Recommendation

*For any* Skill_Profile analyzed for recommendations, the top recommended skill should have an ROI_Score greater than or equal to all other candidate skills considered.

**Validates: Requirements 4.1**

### Property 15: ROI Calculation Comprehensiveness

*For any* ROI_Score calculation, the score should be derived from current salary, potential salary increase, skill acquisition time, and market demand, with changes to any of these inputs resulting in a different ROI_Score.

**Validates: Requirements 4.2**

### Property 16: Recommendation Includes Salary Estimate

*For any* skill recommendation presented to a worker, the recommendation should include a non-zero estimated monthly salary increase value.

**Validates: Requirements 4.3**

### Property 17: Prerequisites Communication

*For any* recommended skill that has defined prerequisites, the recommendation should include a list of those prerequisite skills that must be acquired first.

**Validates: Requirements 4.4**

### Property 18: Learning Time Tie-Breaking

*For any* set of candidate skills with ROI_Scores within 5% of each other, the recommended skill should be the one with the shortest learning time in months.

**Validates: Requirements 4.5**

### Property 19: Job Matching Dual Criteria

*For any* Job_Match returned for a Skill_Profile, the match score calculation should consider both skill alignment (percentage of required skills the worker has) and geographic proximity (distance from worker's location).

**Validates: Requirements 5.1**

### Property 20: Skill Alignment Highlighting

*For any* Job_Match viewed by a worker, the system should identify and mark which of the worker's skills from their Skill_Profile align with the job's required skills.

**Validates: Requirements 5.3**

### Property 21: Interest Triggers Application Info

*For any* Job_Match where a worker expresses interest, the system should provide either application instructions, contact information, or an application URL.

**Validates: Requirements 5.5**

### Property 22: Response Format Flexibility

*For any* system response message, the system should be able to format it as both text and voice (audio), allowing the same content to be delivered in either modality.

**Validates: Requirements 6.3**

### Property 23: Conversation Context Preservation

*For any* conversation session with multiple message exchanges, information extracted in earlier messages (such as skills, location, or language preference) should remain accessible and influence system behavior in later messages within the same session.

**Validates: Requirements 6.5**

### Property 24: PII Encryption at Rest

*For any* personally identifiable information (phone number, name, location) stored in the database, the data should be encrypted, such that direct database queries return encrypted values rather than plaintext.

**Validates: Requirements 8.1**

### Property 25: Consent-Required Data Sharing

*For any* attempt to share worker data with a third party, the operation should only succeed if the worker's profile has an explicit consent flag set to true for that specific third party or purpose.

**Validates: Requirements 8.3**

### Property 26: Data Access Audit Trail

*For any* access to worker data (read, update, or delete operations), an audit log entry should be created containing the timestamp, operation type, data accessed, and accessor identity.

**Validates: Requirements 8.4**

### Property 27: Inconsistency Triggers Clarification

*For any* skill claim that appears inconsistent with stated experience (e.g., claiming 10 years of experience in a technology that's only 3 years old), the system should generate a clarifying question before adding the skill to the Skill_Profile.

**Validates: Requirements 9.2**

### Property 28: Default Self-Reported Status

*For any* newly extracted skill added to a Skill_Profile, the verification status should be set to "SELF_REPORTED" unless external verification data is provided at the time of creation.

**Validates: Requirements 9.3**

### Property 29: Verification Integration

*For any* skill in a Skill_Profile where third-party verification data becomes available, the skill's verification status should be updated to reflect the verification result (VERIFIED or CERTIFIED).

**Validates: Requirements 9.4**

### Property 30: Unverifiable Skill Exclusion

*For any* skill mentioned in conversation that cannot be verified through follow-up questions (worker cannot provide experience duration, project examples, or specific details), the skill should not be added to the final Skill_Profile.

**Validates: Requirements 9.5**

## Error Handling

### Voice Processing Errors

**Transcription Failures**:
- If speech-to-text service fails, retry up to 3 times with exponential backoff
- If all retries fail, send text message asking worker to type their response or try voice again
- Log transcription failures for quality monitoring

**Language Detection Errors**:
- If language cannot be detected with confidence > 0.7, default to asking worker to select language explicitly
- Support language switching at any point in conversation

**Audio Quality Issues**:
- If audio is too short (< 1 second), prompt worker to speak longer
- If audio is too long (> 2 minutes), process in chunks and concatenate results
- If background noise is detected, suggest finding a quieter location

### Skill Extraction Errors

**Ambiguous Skill Mentions**:
- If skill extraction confidence < 0.6, ask clarifying questions
- If worker mentions multiple unrelated skills in one message, ask them to elaborate on each separately
- Maintain a "pending clarification" list for ambiguous skills

**Invalid Skill Categories**:
- If extracted skill doesn't map to known categories, use "OTHER" category and flag for manual review
- Build a feedback loop to improve skill taxonomy over time

**Experience Validation Failures**:
- If experience claims are inconsistent (e.g., total years > age - 15), flag for clarification
- If no experience duration provided, default to "< 1 year" and mark as unverified

### Market Data Errors

**Stale Data**:
- If market data is > 7 days old, trigger background refresh
- If refresh fails, use cached data but inform worker that recommendations are based on recent data
- Never block user interactions waiting for market data refresh

**Missing Location Data**:
- If no market data available for worker's specific city, expand search to state level
- If state-level data unavailable, use national averages with disclaimer
- Always inform worker about the geographic scope of recommendations

**Insufficient Job Listings**:
- If < 10 job listings found for a skill in location, expand search radius by 50km increments
- If still insufficient, show national trends and suggest considering relocation or remote work

### Job Matching Errors

**No Matches Found**:
- If zero exact matches, automatically show near-matches (requiring 1-2 additional skills)
- Provide clear explanation of skill gaps and learning paths
- Suggest skill upgrades that would unlock the most job opportunities

**Expired Listings**:
- Filter out jobs with expiry_date < current_date
- If worker expresses interest in expired job, inform them and suggest similar active listings

### WhatsApp Integration Errors

**Message Delivery Failures**:
- Implement message queue with retry logic (3 attempts, exponential backoff)
- If delivery fails after retries, log error and mark session for manual follow-up
- Don't lose conversation state due to delivery failures

**Rate Limiting**:
- Respect WhatsApp Business API rate limits (80 messages/second)
- Implement token bucket algorithm for rate limiting
- Queue messages if rate limit approached

**Webhook Failures**:
- Implement idempotency for webhook processing (use message_id as idempotency key)
- Handle duplicate webhook deliveries gracefully
- Implement dead letter queue for failed webhook processing

### Data Privacy Errors

**Encryption Failures**:
- If encryption fails during data storage, reject the operation and return error
- Never store PII in plaintext as fallback
- Alert security team immediately on encryption failures

**Consent Violations**:
- If data sharing attempted without consent, reject operation and log security event
- Implement pre-flight consent checks before any external data sharing
- Provide clear audit trail of all consent decisions

### System-Level Errors

**Database Failures**:
- Implement connection pooling with automatic retry
- Use read replicas for query load distribution
- Implement circuit breaker pattern for database calls

**External Service Failures**:
- Implement timeouts for all external API calls (STT, TTS, LLM)
- Use circuit breaker pattern to prevent cascade failures
- Provide graceful degradation (e.g., text-only mode if TTS fails)

**Session Timeout**:
- Set session timeout to 30 minutes of inactivity
- Send reminder message at 25 minutes: "Are you still there?"
- Allow session resumption within 24 hours with context preservation

## Testing Strategy

### Dual Testing Approach

SkillBridge requires both unit testing and property-based testing for comprehensive coverage:

**Unit Tests** focus on:
- Specific examples of skill extraction (e.g., "I've been a plumber for 5 years" → correct skill profile)
- Edge cases (empty audio, network failures, malformed data)
- Integration points (WhatsApp webhook handling, database connections)
- Error conditions (transcription failures, missing data, invalid inputs)

**Property-Based Tests** focus on:
- Universal properties that hold for all inputs (see Correctness Properties section)
- Comprehensive input coverage through randomization
- Invariants that must be maintained (e.g., language consistency, data completeness)
- Round-trip properties (e.g., profile persistence, serialization)

Both approaches are complementary and necessary. Unit tests catch concrete bugs and validate specific scenarios, while property tests verify general correctness across a wide input space.

### Property-Based Testing Configuration

**Framework Selection**:
- **Python**: Use Hypothesis library for property-based testing
- **TypeScript/JavaScript**: Use fast-check library
- **Configuration**: Minimum 100 iterations per property test to ensure adequate randomization coverage

**Test Tagging**:
Each property-based test must include a comment tag referencing the design document property:

```python
# Feature: skillbridge, Property 9: Profile Persistence Round-Trip
@given(skill_profiles())
def test_profile_persistence_roundtrip(profile):
    # Test implementation
    pass
```

**Property Test Structure**:
Each correctness property from the design document must be implemented as a single property-based test. The test should:
1. Generate random valid inputs using the testing framework's generators
2. Execute the system operation
3. Assert that the property holds for the generated inputs
4. Use the framework's shrinking capability to find minimal failing examples

### Test Data Generators

**For Property-Based Tests**, implement generators for:
- Random skill profiles with varying completeness
- Conversation transcripts with different skill mentions
- Market demand data with various distributions
- Job listings with different skill requirements
- Audio buffers with different languages and quality levels
- Geographic locations across India

**Example Generator** (Python/Hypothesis):
```python
from hypothesis import strategies as st

@st.composite
def skill_profiles(draw):
    return SkillProfile(
        profileId=draw(st.uuids()),
        phoneNumber=draw(st.from_regex(r'\+91[0-9]{10}')),
        language=draw(st.sampled_from(['hi', 'ta', 'te', 'bn', 'mr'])),
        skills=draw(st.lists(profiled_skills(), min_size=1, max_size=10)),
        experience=draw(experience_data()),
        location=draw(locations()),
        completenessScore=draw(st.floats(min_value=0, max_value=100)),
        verificationStatus=draw(st.sampled_from(['UNVERIFIED', 'PARTIALLY_VERIFIED', 'VERIFIED'])),
        createdAt=draw(st.datetimes()),
        updatedAt=draw(st.datetimes())
    )
```

### Integration Testing

**WhatsApp Integration**:
- Use WhatsApp Business API sandbox for testing
- Mock webhook deliveries for automated testing
- Test message queuing and retry logic
- Verify rate limiting behavior

**Voice Processing Integration**:
- Use test audio files in all supported languages
- Test with various audio quality levels
- Verify transcription accuracy with known ground truth
- Test language detection with mixed-language audio

**Database Integration**:
- Use test database with realistic data volumes
- Test connection pooling and failover
- Verify encryption at rest
- Test audit logging completeness

**End-to-End Flows**:
- Complete interview flow from greeting to profile creation
- Skill gap analysis and recommendation flow
- Job matching and application flow
- Data deletion and privacy compliance flow

### Performance Testing

**Load Testing**:
- Simulate 1000 concurrent conversations
- Measure response times for voice processing (target: < 10 seconds)
- Measure database query performance (target: < 100ms for profile queries)
- Test WhatsApp API rate limit handling

**Stress Testing**:
- Test system behavior under 10x normal load
- Verify graceful degradation when services fail
- Test recovery after service restoration

### Security Testing

**Data Privacy**:
- Verify PII encryption in database
- Test consent enforcement for data sharing
- Verify audit log completeness
- Test data deletion completeness (no orphaned records)

**Input Validation**:
- Test SQL injection prevention
- Test XSS prevention in text responses
- Verify phone number validation
- Test audio file size limits

### Monitoring and Observability

**Metrics to Track**:
- Conversation completion rate
- Average conversation duration
- Skill extraction accuracy (manual validation sample)
- Transcription error rate
- Job match click-through rate
- Skill recommendation acceptance rate
- System response times (p50, p95, p99)
- Error rates by type

**Logging**:
- Structured logging for all system events
- Correlation IDs for tracing requests across services
- PII redaction in logs
- Log retention: 90 days for operational logs, 7 years for audit logs

**Alerting**:
- Alert on transcription failure rate > 5%
- Alert on conversation abandonment rate > 30%
- Alert on database connection failures
- Alert on WhatsApp API errors
- Alert on encryption failures (critical)
