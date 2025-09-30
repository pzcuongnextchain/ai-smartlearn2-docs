# GPT Analysis Integration Specifications

## Overview
Integration with GPT API for real-time prompt analysis and improvement suggestions in the AI practice tool, as referenced in docs.txt Page 15 with example implementation at https://promptlab-vercel-ready.vercel.app/.

## Integration Requirements from docs.txt

### Core Analysis Features (Page 15)
- **Real-time Analysis**: Analyze prompts entered by students using GPT API
- **Problem Identification**: Provide problem points and improvement suggestions
- **Korean Language Support**: Analysis and feedback in Korean
- **Integration with Practice Tools**: Work alongside AI tool usage

### Analysis Categories (from docs.txt example)
Based on the example shown in docs.txt:
1. **불명확한대상** (Unclear targets)
2. **세부요구없음** (Missing details)  
3. **시간제약누락** (Missing time constraints)
4. **형식미지정** (Format not specified)
5. **안전·윤리조건언급없음** (Safety/ethics conditions not mentioned)

## GPT Analysis Implementation

### Core Analysis Service
```typescript
interface PromptAnalysisRequest {
  prompt: string;
  context?: {
    sessionType: 'practice' | 'assignment';
    toolUsed: string;
    studentLevel: 'beginner' | 'intermediate' | 'advanced';
    organizationId: string;
  };
}

interface PromptAnalysisResponse {
  qualityScore: number; // 0-100
  issues: AnalysisIssue[];
  suggestions: ImprovementSuggestion[];
  strengths: string[];
  categories: {
    clarity: 'good' | 'needs_improvement' | 'poor';
    specificity: 'good' | 'needs_improvement' | 'poor';
    format: 'specified' | 'partial' | 'missing';
    ethics: 'mentioned' | 'implied' | 'not_mentioned';
    timeConstraints: 'clear' | 'vague' | 'missing';
  };
}

class GPTPromptAnalyzer {
  constructor(
    private apiKey: string,
    private organizationId: string
  ) {}
  
  async analyzePrompt(request: PromptAnalysisRequest): Promise<PromptAnalysisResponse> {
    const analysisPrompt = this.buildAnalysisPrompt(request);
    
    try {
      const gptResponse = await this.callGPTAPI(analysisPrompt);
      return this.parseAnalysisResponse(gptResponse);
    } catch (error) {
      console.error('GPT Analysis failed:', error);
      return this.getFallbackAnalysis(request.prompt);
    }
  }
  
  private buildAnalysisPrompt(request: PromptAnalysisRequest): string {
    return `
당신은 AI 학습 도구를 위한 프롬프트 분석 전문가입니다. 
다음 프롬프트를 분석하고 개선 제안을 해주세요:

프롬프트: "${request.prompt}"

다음 기준으로 분석해주세요:
1. 불명확한 대상: 대상이 명확하게 지정되었는가?
2. 세부 요구사항: 구체적인 요구사항이 포함되었는가?
3. 시간 제약: 시간 관련 제약이나 기한이 명시되었는가?
4. 형식 지정: 원하는 출력 형식이 지정되었는가?
5. 안전·윤리 조건: 안전하고 윤리적인 조건이 고려되었는가?

JSON 형식으로 응답해주세요:
{
  "score": 점수(0-100),
  "issues": [
    {
      "category": "카테고리",
      "severity": "high|medium|low", 
      "description": "문제 설명",
      "example": "개선 예시"
    }
  ],
  "suggestions": [
    {
      "type": "improvement_type",
      "description": "개선 제안",
      "example": "개선된 프롬프트 예시"
    }
  ],
  "strengths": ["잘된 점들"],
  "categories": {
    "clarity": "good|needs_improvement|poor",
    "specificity": "good|needs_improvement|poor", 
    "format": "specified|partial|missing",
    "ethics": "mentioned|implied|not_mentioned",
    "timeConstraints": "clear|vague|missing"
  }
}
`;
  }
}
```

### Real-time Analysis Pipeline
```typescript
class RealTimeAnalysisService {
  private analysisQueue: Map<string, PromptAnalysisRequest> = new Map();
  private debounceTimer: NodeJS.Timeout | null = null;
  
  constructor(
    private gptAnalyzer: GPTPromptAnalyzer,
    private uiUpdater: AnalysisUIUpdater
  ) {}
  
  queueAnalysis(promptId: string, request: PromptAnalysisRequest): void {
    // Add to queue and debounce to avoid excessive API calls
    this.analysisQueue.set(promptId, request);
    
    if (this.debounceTimer) {
      clearTimeout(this.debounceTimer);
    }
    
    this.debounceTimer = setTimeout(() => {
      this.processAnalysisQueue();
    }, 1000); // 1 second debounce
  }
  
  private async processAnalysisQueue(): Promise<void> {
    const requests = Array.from(this.analysisQueue.entries());
    this.analysisQueue.clear();
    
    // Process most recent request for each prompt
    const latestRequests = requests.reduce((acc, [id, request]) => {
      acc.set(id, request);
      return acc;
    }, new Map<string, PromptAnalysisRequest>());
    
    // Analyze in parallel
    const analyses = await Promise.allSettled(
      Array.from(latestRequests.entries()).map(async ([id, request]) => {
        const analysis = await this.gptAnalyzer.analyzePrompt(request);
        return { id, analysis };
      })
    );
    
    // Update UI with results
    analyses.forEach(result => {
      if (result.status === 'fulfilled') {
        this.uiUpdater.updateAnalysis(result.value.id, result.value.analysis);
      }
    });
  }
}
```

### Analysis UI Integration
```typescript
// Integration with Assistant UI for displaying analysis
class AnalysisUIUpdater {
  constructor(private analysisPanel: AnalysisPanelComponent) {}
  
  updateAnalysis(promptId: string, analysis: PromptAnalysisResponse): void {
    this.analysisPanel.setState({
      currentAnalysis: analysis,
      isLoading: false,
      lastUpdated: new Date()
    });
    
    // Update analysis display
    this.displayQualityScore(analysis.qualityScore);
    this.displayIssues(analysis.issues);
    this.displaySuggestions(analysis.suggestions);
    this.displayStrengths(analysis.strengths);
  }
  
  private displayQualityScore(score: number): void {
    const scoreElement = document.getElementById('quality-score');
    if (scoreElement) {
      scoreElement.textContent = score.toString();
      scoreElement.className = this.getScoreClass(score);
    }
  }
  
  private displayIssues(issues: AnalysisIssue[]): void {
    const issuesContainer = document.getElementById('analysis-issues');
    if (!issuesContainer) return;
    
    issuesContainer.innerHTML = issues.map(issue => `
      <div class="issue-item severity-${issue.severity}">
        <h4>${issue.category}</h4>
        <p>${issue.description}</p>
        ${issue.example ? `<div class="example">${issue.example}</div>` : ''}
      </div>
    `).join('');
  }
  
  private displaySuggestions(suggestions: ImprovementSuggestion[]): void {
    const suggestionsContainer = document.getElementById('improvement-suggestions');
    if (!suggestionsContainer) return;
    
    suggestionsContainer.innerHTML = suggestions.map(suggestion => `
      <div class="suggestion-item">
        <h4>${suggestion.type}</h4>
        <p>${suggestion.description}</p>
        ${suggestion.example ? `
          <div class="suggestion-example">
            <strong>개선 예시:</strong>
            <code>${suggestion.example}</code>
          </div>
        ` : ''}
      </div>
    `).join('');
  }
}
```

## Analysis Categories Implementation

### Issue Detection Algorithms
```typescript
class IssueDetector {
  detectUnclearTargets(prompt: string): AnalysisIssue | null {
    // Check for vague pronouns, unclear references
    const vaguePatterns = [
      /이것을?/g, /그것을?/g, /저것을?/g,
      /뭔가/g, /어떤 것/g, /무언가/g
    ];
    
    const hasVagueReferences = vaguePatterns.some(pattern => 
      pattern.test(prompt)
    );
    
    if (hasVagueReferences) {
      return {
        category: '불명확한 대상',
        severity: 'medium',
        description: '대상이 명확하지 않습니다. 구체적인 대상을 지정해주세요.',
        example: '"이것을 분석해줘" → "다음 마케팅 전략을 분석해줘"'
      };
    }
    
    return null;
  }
  
  detectMissingDetails(prompt: string): AnalysisIssue | null {
    // Check for lack of specific requirements
    const wordCount = prompt.split(' ').length;
    const hasSpecifics = /구체적|자세히|상세히|정확히/.test(prompt);
    
    if (wordCount < 10 && !hasSpecifics) {
      return {
        category: '세부 요구사항 부족',
        severity: 'high',
        description: '더 구체적인 요구사항을 추가해주세요.',
        example: '"글을 써줘" → "마케팅 제품 소개를 위한 500자 내외의 블로그 글을 써줘"'
      };
    }
    
    return null;
  }
  
  detectMissingTimeConstraints(prompt: string): AnalysisIssue | null {
    const timePatterns = [
      /\d+분/, /\d+시간/, /\d+일/, /빠르게/, /즉시/, 
      /마감/, /기한/, /언제까지/
    ];
    
    const hasTimeReference = timePatterns.some(pattern => 
      pattern.test(prompt)
    );
    
    if (!hasTimeReference && prompt.includes('계획') || prompt.includes('일정')) {
      return {
        category: '시간 제약 누락',
        severity: 'medium',
        description: '시간 제약이나 기한을 명시해주세요.',
        example: '"계획을 세워줘" → "다음 주까지 완료할 수 있는 계획을 세워줘"'
      };
    }
    
    return null;
  }
  
  detectMissingFormat(prompt: string): AnalysisIssue | null {
    const formatPatterns = [
      /표/, /목록/, /리스트/, /JSON/, /CSV/, /형식/, 
      /포맷/, /양식/, /템플릿/
    ];
    
    const hasFormatRequest = formatPatterns.some(pattern => 
      pattern.test(prompt)
    );
    
    const needsFormat = prompt.includes('정리') || prompt.includes('요약');
    
    if (needsFormat && !hasFormatRequest) {
      return {
        category: '형식 미지정',
        severity: 'low',
        description: '원하는 출력 형식을 지정해주세요.',
        example: '"정리해줘" → "표 형식으로 정리해줘" 또는 "번호가 있는 목록으로 정리해줘"'
      };
    }
    
    return null;
  }
}
```

### Improvement Suggestion Generator
```typescript
class SuggestionGenerator {
  generateImprovements(prompt: string, issues: AnalysisIssue[]): ImprovementSuggestion[] {
    const suggestions: ImprovementSuggestion[] = [];
    
    // Generate specific suggestions based on detected issues
    issues.forEach(issue => {
      switch (issue.category) {
        case '불명확한 대상':
          suggestions.push({
            type: 'clarity',
            description: '구체적인 대상을 명시하여 명확성을 높이세요.',
            example: this.generateClarityExample(prompt)
          });
          break;
          
        case '세부 요구사항 부족':
          suggestions.push({
            type: 'specificity',
            description: '더 구체적인 요구사항과 조건을 추가하세요.',
            example: this.generateSpecificityExample(prompt)
          });
          break;
          
        case '형식 미지정':
          suggestions.push({
            type: 'format',
            description: '원하는 출력 형식을 명확히 지정하세요.',
            example: this.generateFormatExample(prompt)
          });
          break;
      }
    });
    
    return suggestions;
  }
  
  private generateClarityExample(prompt: string): string {
    // Generate context-aware clarity improvements
    if (prompt.includes('분석')) {
      return `"${prompt}" → "다음 데이터를 마케팅 관점에서 분석해주세요: [구체적 데이터]"`;
    }
    
    return `더 구체적인 대상을 지정: "${prompt}" → "[구체적 대상]에 대해 ${prompt}"`;
  }
  
  private generateSpecificityExample(prompt: string): string {
    return `"${prompt}" → "${prompt} (대상: [구체적 대상], 목적: [목적], 조건: [조건])"`;
  }
  
  private generateFormatExample(prompt: string): string {
    return `"${prompt}" → "${prompt} (형식: 표/목록/단계별로 정리)"`;
  }
}
```

## Performance and Caching

### Analysis Caching Strategy
```typescript
class AnalysisCache {
  private cache = new Map<string, CachedAnalysis>();
  private readonly TTL = 5 * 60 * 1000; // 5 minutes
  
  getCachedAnalysis(promptHash: string): PromptAnalysisResponse | null {
    const cached = this.cache.get(promptHash);
    
    if (cached && Date.now() - cached.timestamp < this.TTL) {
      return cached.analysis;
    }
    
    // Remove expired cache
    if (cached) {
      this.cache.delete(promptHash);
    }
    
    return null;
  }
  
  setCachedAnalysis(promptHash: string, analysis: PromptAnalysisResponse): void {
    this.cache.set(promptHash, {
      analysis,
      timestamp: Date.now()
    });
    
    // Cleanup old entries
    this.cleanupExpiredEntries();
  }
  
  private cleanupExpiredEntries(): void {
    const now = Date.now();
    for (const [key, value] of this.cache.entries()) {
      if (now - value.timestamp >= this.TTL) {
        this.cache.delete(key);
      }
    }
  }
}
```

### Rate Limiting and Cost Control
```typescript
class GPTAnalysisRateLimiter {
  private requestCounts = new Map<string, number>();
  private readonly MAX_REQUESTS_PER_MINUTE = 30;
  
  canMakeRequest(organizationId: string): boolean {
    const currentCount = this.requestCounts.get(organizationId) || 0;
    return currentCount < this.MAX_REQUESTS_PER_MINUTE;
  }
  
  recordRequest(organizationId: string): void {
    const currentCount = this.requestCounts.get(organizationId) || 0;
    this.requestCounts.set(organizationId, currentCount + 1);
    
    // Reset counter after 1 minute
    setTimeout(() => {
      this.requestCounts.set(organizationId, Math.max(0, 
        (this.requestCounts.get(organizationId) || 0) - 1
      ));
    }, 60000);
  }
}
```

## Integration with Practice Tools

### Synchronized Analysis Display
```typescript
// Display analysis alongside AI tool responses
class SynchronizedAnalysisDisplay {
  constructor(
    private assistantUI: AssistantUIComponent,
    private analysisPanel: AnalysisPanelComponent
  ) {}
  
  async displayPromptWithAnalysis(
    prompt: string, 
    toolResponse: AIToolResponse,
    analysis: PromptAnalysisResponse
  ): Promise<void> {
    // Display AI tool response in main chat
    this.assistantUI.addMessage({
      role: 'assistant',
      content: toolResponse.content,
      timestamp: new Date()
    });
    
    // Display analysis in side panel
    this.analysisPanel.updateAnalysis({
      prompt,
      analysis,
      toolUsed: toolResponse.toolId,
      timestamp: new Date()
    });
    
    // Highlight areas for improvement
    this.highlightImprovementAreas(analysis.issues);
  }
  
  private highlightImprovementAreas(issues: AnalysisIssue[]): void {
    issues.forEach(issue => {
      if (issue.severity === 'high') {
        this.showImprovementNotification(issue);
      }
    });
  }
}
```

## Success Metrics and Monitoring

### Analysis Quality Metrics
- Analysis response time < 2 seconds
- Analysis accuracy rate > 85% (based on user feedback)
- Cache hit rate > 60% for similar prompts
- User engagement with suggestions > 70%
- Prompt quality improvement measurable over time

### Cost and Usage Monitoring
```typescript
class AnalysisUsageMonitor {
  trackAnalysisRequest(organizationId: string, cost: number): void {
    analytics.track('gpt_analysis_request', {
      organizationId,
      cost,
      timestamp: new Date(),
      cacheHit: false
    });
  }
  
  trackCacheHit(organizationId: string): void {
    analytics.track('gpt_analysis_cache_hit', {
      organizationId,
      cost: 0,
      timestamp: new Date(),
      cacheHit: true
    });
  }
  
  generateUsageReport(organizationId: string): AnalysisUsageReport {
    return {
      totalRequests: this.getTotalRequests(organizationId),
      totalCost: this.getTotalCost(organizationId),
      cacheHitRate: this.getCacheHitRate(organizationId),
      averageResponseTime: this.getAverageResponseTime(organizationId)
    };
  }
}
```

## Dependencies and Integration Points
- OpenAI GPT API for prompt analysis
- Assistant UI integration for display
- Caching system for performance optimization
- Rate limiting for cost control
- Analytics system for usage monitoring
- Real-time WebSocket for immediate feedback