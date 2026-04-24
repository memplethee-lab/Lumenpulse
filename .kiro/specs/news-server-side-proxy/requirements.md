# Requirements Document

## Introduction

The webapp currently fetches news directly from the client side with a hardcoded NewsAPI key exposed in the frontend code. This creates security vulnerabilities and violates API key best practices. This feature will move the news fetching flow behind a secure server-side route or backend proxy while preserving the existing news user experience.

## Glossary

- **Webapp**: The Next.js frontend application located in `apps/webapp`
- **Backend**: The NestJS backend API service located in `apps/backend`
- **NewsAPI**: Third-party news API service (newsapi.org) currently used by the webapp
- **CoinDesk API**: Third-party news API service currently used by the backend
- **API Key**: Secret credential used to authenticate with external API services
- **Client-side**: Code that executes in the user's browser
- **Server-side**: Code that executes on the backend server
- **News UX**: The user interface and experience for browsing and viewing news articles
- **API Proxy**: A server-side endpoint that forwards requests to external APIs

## Requirements

### Requirement 1

**User Story:** As a developer, I want API keys to be stored securely on the server, so that they are not exposed in client-side code or network requests.

#### Acceptance Criteria

1. WHEN the webapp needs to fetch news THEN the system SHALL make requests to a backend API endpoint instead of directly to external news providers
2. WHEN the backend receives a news request THEN the system SHALL use server-side stored API keys to fetch data from external providers
3. WHEN a user inspects the webapp source code or network requests THEN the system SHALL NOT expose any API keys
4. WHEN the backend makes external API requests THEN the system SHALL include API keys only in server-side HTTP headers
5. THE backend SHALL store all API keys in environment variables and SHALL NOT hardcode them in source code

### Requirement 2

**User Story:** As a user, I want to continue seeing crypto news articles with the same interface and functionality, so that my experience is not disrupted by backend changes.

#### Acceptance Criteria

1. WHEN a user visits the news page THEN the system SHALL display news articles in the same grid layout as before
2. WHEN news articles are displayed THEN the system SHALL show title, excerpt, author, date, image, category, sentiment, and funding status
3. WHEN a user clicks on a news article THEN the system SHALL open the article URL in a new tab
4. WHEN news data is loading THEN the system SHALL display the same loading skeleton UI
5. WHEN news data fails to load THEN the system SHALL fall back to the AI-generated news as before

### Requirement 3

**User Story:** As a user, I want to filter and sort news articles, so that I can find relevant content quickly.

#### Acceptance Criteria

1. WHEN a user selects a category filter THEN the system SHALL display only articles matching that category
2. WHEN a user selects a sentiment filter THEN the system SHALL display only articles matching that sentiment
3. WHEN a user selects a funding status filter THEN the system SHALL display only articles matching that funding status
4. WHEN a user changes the sort order THEN the system SHALL reorder articles according to the selected criteria
5. WHEN filters are applied THEN the system SHALL display the count of filtered results

### Requirement 4

**User Story:** As a developer, I want the backend to provide a unified news API endpoint, so that the frontend has a consistent interface regardless of the news provider.

#### Acceptance Criteria

1. THE backend SHALL provide a GET endpoint at `/api/news` that returns news articles
2. WHEN the frontend requests news THEN the backend SHALL return articles in a standardized format
3. THE backend SHALL accept query parameters for limit, category, tag, and other filters
4. WHEN the backend fetches news from external providers THEN the system SHALL transform the response into a consistent format
5. THE backend SHALL handle errors from external providers and return appropriate HTTP status codes

### Requirement 5

**User Story:** As a developer, I want to leverage the existing backend news infrastructure, so that I don't duplicate functionality.

#### Acceptance Criteria

1. THE system SHALL use the existing NestJS news module located in `apps/backend/src/news`
2. WHEN the backend needs to fetch news THEN the system SHALL use the existing NewsProviderService
3. THE system SHALL reuse existing news DTOs and interfaces for data transformation
4. THE system SHALL leverage existing caching mechanisms for news data
5. THE system SHALL maintain compatibility with existing news database entities

### Requirement 6

**User Story:** As a user, I want news to load quickly, so that I can access information without delays.

#### Acceptance Criteria

1. WHEN news is requested THEN the backend SHALL return cached data if available and not expired
2. THE backend SHALL cache news responses for at least 5 minutes
3. WHEN cached data is served THEN the response time SHALL be under 200 milliseconds
4. WHEN fresh data is fetched THEN the backend SHALL update the cache
5. THE system SHALL invalidate cache when new articles are added to the database

### Requirement 7

**User Story:** As a developer, I want the webapp to gracefully handle backend unavailability, so that users still see content when the backend is down.

#### Acceptance Criteria

1. WHEN the backend is unavailable THEN the webapp SHALL fall back to the AI-generated news component
2. WHEN a backend request times out THEN the webapp SHALL display an error message and fallback content
3. WHEN the backend returns an error status THEN the webapp SHALL log the error and show fallback content
4. THE webapp SHALL set a request timeout of 10 seconds for news API calls
5. WHEN fallback content is displayed THEN the system SHALL indicate to users that content is AI-generated

### Requirement 8

**User Story:** As a developer, I want to remove the hardcoded NewsAPI key from the webapp, so that the codebase is secure and follows best practices.

#### Acceptance Criteria

1. THE system SHALL remove the NewsApiService class from `apps/webapp/lib/api-services.ts`
2. THE system SHALL remove all references to the hardcoded NEWS_API_KEY constant
3. THE system SHALL remove all direct calls to newsapi.org from the webapp
4. WHEN the webapp needs news data THEN the system SHALL call the backend API endpoint
5. THE system SHALL update all components that use NewsApiService to use the new backend endpoint

### Requirement 9

**User Story:** As a developer, I want the webapp to use environment variables for backend API configuration, so that the backend URL can be changed without code modifications.

#### Acceptance Criteria

1. THE webapp SHALL read the backend API base URL from an environment variable
2. WHEN the environment variable is not set THEN the system SHALL use a default localhost URL for development
3. THE webapp SHALL construct full API URLs by combining the base URL with endpoint paths
4. THE system SHALL support different backend URLs for development, staging, and production environments
5. THE webapp SHALL validate that the backend URL is properly formatted before making requests
