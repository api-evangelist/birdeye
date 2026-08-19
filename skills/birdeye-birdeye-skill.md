---
name: Birdeye
description: Use when building integrations with Birdeye's reputation management platform, querying review data, managing business listings, analyzing customer feedback, scheduling social posts, or connecting AI assistants to Birdeye account data via MCP.
metadata:
    mintlify-proj: birdeye
    version: "1.0"
---

# Birdeye Skill

## Product Summary

Birdeye is a reputation management and customer experience platform that helps businesses manage reviews, listings, surveys, social media, and customer feedback across multiple locations. Agents work with Birdeye through two primary interfaces: the **REST API** (for backend integrations and data operations) and the **MCP Server** (for AI assistant connections). The REST API requires an `x-api-key` header for authentication and uses resource-oriented endpoints. The MCP Server at `https://mcp.birdeye.com/mcp` uses OAuth 2.0 and connects AI assistants (Claude, ChatGPT) to live Birdeye account data without manual token management. Key file paths: API docs at https://docs.birdeye.com/api, MCP docs at https://docs.birdeye.com/mcp.

## When to Use

Reach for this skill when:

- **Building backend integrations**: Creating systems that read/write reviews, manage business profiles, handle contacts, or sync listings
- **Querying account data via AI**: Connecting Claude or ChatGPT to Birdeye to answer questions about reviews, ratings, listings, surveys, or competitor data
- **Managing multi-location operations**: Fetching child locations, aggregating metrics across locations, or updating business information at scale
- **Analyzing reputation metrics**: Pulling review trends, sentiment analysis, Search AI accuracy reports, or competitive benchmarks
- **Automating social media**: Scheduling posts, tracking performance, or uploading media to social channels
- **Handling customer feedback**: Creating surveys, managing contacts, or tracking customer experience scores
- **Setting up webhooks**: Subscribing to real-time events like new reviews, survey responses, or ticket updates

## Quick Reference

### Authentication

| Method | Use Case | Details |
|--------|----------|---------|
| **REST API** | Backend integrations | Requires `x-api-key` header; fetch from Birdeye dashboard; keep server-side only |
| **MCP OAuth** | AI assistant connections | OAuth 2.0 with Dynamic Client Registration; no manual token management; pre-authorized for Claude and ChatGPT |

### Common HTTP Status Codes

| Code | Meaning | Action |
|------|---------|--------|
| 200 | Success | Request completed |
| 202 | Accepted | Async operation queued (check status later) |
| 400 | Bad Request | Invalid parameter or missing required field |
| 401 | Unauthorized | Invalid/missing API key or expired OAuth token |
| 404 | Not Found | Resource doesn't exist |
| 429 | Rate Limited | Too many requests; wait before retrying |
| 500 | Server Error | Contact support |

### Pagination

Use `sindex` (start index) and `count` (number of records) for paginated endpoints:

```
sindex=0, count=10    # First 10 records
sindex=10, count=10   # Next 10 records
```

**Limit**: `sindex + count` must not exceed 100,000. Use filters to narrow results if needed.

### MCP Tools by Category

| Category | Key Tools | Purpose |
|----------|-----------|---------|
| **Reviews** | `get_reviews`, `review_and_rating_overview`, `get_review_summary` | Fetch individual reviews, trends, and aggregated ratings |
| **Business** | `get_business_info`, `get_child_locations` | Get account profile and list all locations |
| **Listings** | `get_listing`, `get_listing_location_status_report`, `get_listing_insights` | Retrieve listing profiles, sync status, and analytics |
| **Search AI** | `get_search_ai_businesses`, `get_search_ai_accuracy_report`, `get_search_ai_sentiment_report` | Query AI engine visibility, NAP accuracy, SWOT analysis |
| **Surveys** | `get_all_surveys`, `get_survey_responses` | List surveys and fetch paginated responses |
| **Social** | `get_social_open_url_performance_report` | Track social channel performance |
| **Ticketing** | `get_all_ticket_data` | Retrieve tickets with filtering and pagination |
| **Competitor AI** | `retrieve_competitor_reviews`, `retrieve_competitor_review_metrics` | Analyze competitor reviews and ratings |
| **Insight AI** | `get_insight_experience_score_benchmark`, `get_insight_experience_location_info` | Compare experience scores vs industry benchmarks |

### REST API Endpoints (Common)

| Endpoint | Method | Purpose |
|----------|--------|---------|
| `/v1/business/{businessId}` | GET/PUT | Get or update business info |
| `/v1/business` | POST | Create new business |
| `/v1/review/get` | POST | Fetch reviews with filters |
| `/v1/reports/review/analytic/time` | POST | Get review trends over time |
| `/v1/listing/status/location` | GET | Check listing sync status |
| `/v1/contact/details` | POST | List customers/leads |
| `/v1/customer-v2/external/upsertCustomer` | POST | Create or update contact |
| `/v1/social/post/public/schedule` | POST | Schedule social post |
| `/v1/survey/ext/list` | GET | List surveys |
| `/v1/subscriptions/subscribe` | POST | Create webhook subscription |

## Decision Guidance

### When to Use REST API vs MCP

| Scenario | Use REST API | Use MCP |
|----------|-------------|---------|
| Backend system integration | ✓ | — |
| AI assistant connection | — | ✓ |
| Writing/modifying data | ✓ | — (read-only) |
| Real-time queries from AI | — | ✓ |
| Webhook subscriptions | ✓ | — |
| Multi-location aggregation | ✓ | ✓ |

### When to Use Contact V1 vs Contact V2

| Feature | Contact V1 | Contact V2 |
|---------|-----------|-----------|
| Basic CRUD | ✓ | ✓ |
| Communication preferences | — | ✓ |
| Opt-out management | ✓ | ✓ |
| Simpler integration | ✓ | — |

## Workflow

### Typical Task: Query Reviews via MCP

1. **Connect AI client**: Add `https://mcp.birdeye.com/mcp` to Claude or ChatGPT settings
2. **Authorize**: Sign in with Birdeye credentials (one-time OAuth flow)
3. **Call `get_business_info`**: Verify account access and get business name
4. **Call `get_child_locations`**: If enterprise account, retrieve location IDs
5. **Call `get_reviews`** or **`review_and_rating_overview`**: Query reviews with date range, source filters, or rating filters
6. **Parse response**: Extract review text, ratings, dates, and reviewer info
7. **Verify**: Confirm data matches expected date range and location scope

### Typical Task: Create/Update Business via REST API

1. **Get API key**: Fetch from Birdeye dashboard; store securely server-side
2. **Prepare request body**: Gather business name, address, phone, email, timezone, hours, etc.
3. **Call POST `/v1/signup/reseller/subaccount`** (create) or **PUT `/v1/business/{businessId}`** (update)
4. **Include headers**: `x-api-key`, `Content-Type: application/json`, `Accept: application/json`
5. **Handle response**: Check HTTP status; extract `businessId` from 200 response
6. **Verify**: Call GET `/v1/business/{businessId}` to confirm changes persisted

### Typical Task: Schedule Social Post

1. **Prepare content**: Text, media URLs, social site (Google, Facebook, Instagram, LinkedIn, Twitter)
2. **Get location IDs**: Call `get_child_locations` or use known business numbers
3. **Call POST `/v1/social/post/public/schedule`**: Include text, media, schedule time, location IDs
4. **Capture tracking ID**: Store from response for status tracking
5. **Track status**: Poll GET `/v1/social/{accountNumber}/post/track/{trackingId}` to monitor publishing
6. **Verify**: Confirm post appears on social channel at scheduled time

### Typical Task: Set Up Webhook

1. **Identify events**: Choose from available webhook events (e.g., `review_created`, `survey_response_submitted`)
2. **Prepare endpoint**: Create HTTPS endpoint on your server to receive POST requests
3. **Call POST `/v1/subscriptions/subscribe`**: Include webhook URL, business ID, event name, optional auth
4. **Store subscription ID**: Save for future unsubscribe operations
5. **Test**: Trigger event and verify webhook receives payload
6. **Monitor**: Log incoming webhooks; handle retries if endpoint is slow

## Common Gotchas

- **API key exposure**: Never expose `x-api-key` in client-side code or version control. Always call REST APIs from backend servers only.
- **OAuth token expiry**: MCP tokens have limited lifetime. Most AI clients auto-refresh; if not, disconnect and reconnect the MCP server.
- **Deep pagination limit**: `sindex + count` cannot exceed 100,000. Use date filters or source filters to reduce result set before paginating.
- **Location ID requirement**: Many tools require `locationId` or `businessNumber`. Always call `get_child_locations` first for enterprise accounts.
- **Timezone format**: Business timezone must match IANA format (e.g., `America/New_York`). Invalid timezones cause 400 errors.
- **Date format inconsistency**: Some endpoints expect `MM/DD/YYYY`, others `yyyy-MM-dd`. Check endpoint docs carefully.
- **Reviewer email redaction**: Email addresses are automatically filtered from review responses for privacy; don't expect them in API responses.
- **Webhook retry logic**: Birdeye retries failed webhook deliveries. Ensure your endpoint is idempotent (safe to receive duplicate events).
- **Rate limiting**: Each API key has a request limit. If you hit 429, back off exponentially before retrying.
- **MCP read-only**: MCP tools cannot write data. Use REST API for create/update/delete operations.
- **Missing required headers**: REST API calls without `x-api-key` or with wrong `Content-Type` will fail with 400/401.

## Verification Checklist

Before submitting work:

- [ ] **API key**: Confirmed it's stored server-side, not in client code or logs
- [ ] **Authentication**: Tested with valid credentials; confirmed 200/202 responses, not 401
- [ ] **Required fields**: Verified all required parameters are present in request body
- [ ] **Date formats**: Confirmed dates match endpoint specification (MM/DD/YYYY vs yyyy-MM-dd)
- [ ] **Location scope**: For multi-location operations, confirmed correct `businessNumber` or `locationId` is used
- [ ] **Pagination**: If fetching large datasets, confirmed `sindex + count ≤ 100,000`
- [ ] **Response parsing**: Verified response structure matches documentation; handled null/missing fields
- [ ] **Error handling**: Confirmed 400/401/429 responses are caught and logged
- [ ] **Webhook endpoint**: If using webhooks, confirmed endpoint is HTTPS, publicly accessible, and idempotent
- [ ] **MCP vs REST**: Confirmed correct interface is used (MCP for AI queries, REST for writes)
- [ ] **Data privacy**: Confirmed sensitive fields (emails, phone numbers) are handled per Birdeye's redaction rules

## Resources

- **Comprehensive page listing**: https://docs.birdeye.com/llms.txt
- **API Reference**: https://docs.birdeye.com/api/introduction
- **MCP Server Guide**: https://docs.birdeye.com/mcp/introduction
- **Authentication Details**: https://docs.birdeye.com/api/authentication (REST) and https://docs.birdeye.com/mcp/authentication (MCP)

---

> For additional documentation and navigation, see: https://docs.birdeye.com/llms.txt