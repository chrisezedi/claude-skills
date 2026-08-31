---
name: writing-tests
description: Use when writing tests for existing code, adding test coverage, or creating test files — particularly when the goal is behavioral testing, mocking class dependencies, or avoiding implementation-detail assertions in TypeScript/NestJS projects.
---

# Writing Tests

## Core Principle

**Test what the code produces for its callers, not how it produces it.**

A test that breaks when you rename a private method is not testing behavior — it's testing implementation. Tests should survive refactoring.

## What to Test vs What to Skip

| Test this | Skip this |
|-----------|-----------|
| Messages published to external systems | Which internal method was called |
| Data written to the database | Exact number of times a method was called |
| Errors thrown | Logger/observability calls |
| Timestamps and state updated | Internal channel routing logic |
| Correct data flows through | Argument delegation to collaborators |

## Test Naming

`describe` blocks are named `ServiceName.methodName`. `it` blocks read as behavior specs.

```typescript
// ❌ Vague journey name / implementation assertion
describe('User signs up', () => {
  it('calls db.user.create', ...)
})

// ✅ Method name / behavior spec
describe('UserService.create', () => {
  it('creates and returns the new user', ...)
})

describe('InsightsService.getAdsLeaderboard', () => {
  it('groups ads sharing the same creative into a single entry', ...)
  it('returns CPC as 0 when there are no link clicks', ...)
})
```

## Mocking Class Dependencies

When a class creates external connections (Redis, HTTP clients) in its constructor:

**Step 1** — Mock the module so the constructor does not fail:
```typescript
jest.mock('ioredis', () => ({
  Redis: jest.fn().mockImplementation(() => ({ publish: jest.fn() })),
}));
```

**Step 2** — After instantiation, replace the instance property with a spy:
```typescript
(processor as unknown as { redisClient: { publish: jest.Mock } }).redisClient = {
  publish: mockRedisPublish,
};
```

Use `as unknown as { ... }` — never `as any`. The typed cast documents the shape and avoids lint errors.

## Typing Mocks

Define types for callbacks and complex structures at the top of the file:

```typescript
type ActivityPage = { eventTime: string }[];
type FetchActivitiesCallback = (page: ActivityPage) => Promise<void>;
```

Use them in `mockImplementation` to avoid `@typescript-eslint/no-unsafe-call` errors:

```typescript
mockFetchActivities.mockImplementation(
  async (
    _id: string,
    _token: string,
    _date: Date | null,
    _tz: string | null,
    callback: FetchActivitiesCallback,
  ) => {
    await callback(page);
  },
);
```

## Typed Message Helpers

When asserting on messages from a union type, use a type guard instead of casting:

```typescript
type PublishedMessage = SyncStageMessage | SyncCompletionMessage | SyncFailureMessage;

const isSyncStageMessage = (m: PublishedMessage): m is SyncStageMessage =>
  'syncStage' in m;

// Then filter cleanly:
publishedMessages()
  .filter(isSyncStageMessage)
  .filter((m) => m.status === 'synced')
  .map((m) => m.monthSynced); // fully typed, no lint errors
```

## Shared Helpers

Extract repeated setup at `describe` scope so each test stays focused:

```typescript
const makeJob = (data = jobData): Job =>
  ({ id: 'job-1', data }) as unknown as Job;

const publishedMessages = (): PublishedMessage[] =>
  mockRedisPublish.mock.calls.map(
    ([, msg]: [string, string]) => JSON.parse(msg) as PublishedMessage,
  );
```

## Two Types of Tests

Settle on two test types and know when to use each:

| Type | Layer | Tools | When to use |
|------|-------|-------|-------------|
| **Behavior** | Service | `@nestjs/testing`, jest mocks | Business logic, data transformations, error paths |
| **Wiring** | HTTP | `supertest`, real modules | Critical endpoints only — verifies guards, pipes, and the full pipeline connect |

Controller tests are usually not worth writing — they are thin wrappers with no business logic. Write behavior tests for services; write wiring tests for 2–3 critical endpoints.

## NestJS Service Test Setup

```typescript
// Type your mocks — never use `any`
type MockDb = {
  user: { create: jest.Mock; findUnique: jest.Mock; update: jest.Mock };
  integration: { findUnique: jest.Mock };
};

describe('UserService', () => {
  let service: UserService;
  let db: MockDb;

  beforeEach(async () => {
    db = {
      user: { create: jest.fn(), findUnique: jest.fn(), update: jest.fn() },
      integration: { findUnique: jest.fn() },
    };

    const module: TestingModule = await Test.createTestingModule({
      providers: [
        UserService,
        { provide: DatabaseService, useValue: db },
        // BullMQ queues need getQueueToken
        { provide: getQueueToken('queue-name'), useValue: { add: jest.fn() } },
      ],
    }).compile();

    service = module.get<UserService>(UserService);
  });
});
```

## Prisma Decimal Mocking

Prisma stores monetary values as `Decimal`. Mock them with a plain object:

```typescript
const decimal = (value: number) => ({ toNumber: () => value });

// Use in mock data:
{ spend: decimal(100), cpm: decimal(0), cpc: decimal(0) }
```

## Filter/DTO Helpers

When a method accepts a DTO with many required fields, create a helper that fills defaults so tests only specify the field they care about:

```typescript
const filter = (overrides: Partial<AdInsightsFilters>): AdInsightsFilters => ({
  since: '',
  until: '',
  limit: 20,
  campaignObjective: [] as CampaignObjective[],
  ...overrides,
});

// In tests:
await service.getAdsLeaderboard(accountId, filter({ effective_status: ['ACTIVE'] }));
```

## Avoiding `no-unsafe-assignment` on mock.calls

When `expect.objectContaining` inside `toHaveBeenCalledWith` triggers lint errors, destructure `mock.calls` directly instead:

```typescript
// ❌ Triggers @typescript-eslint/no-unsafe-assignment
expect(db.user.update).toHaveBeenCalledWith(
  expect.objectContaining({ data: expect.objectContaining({ metaData: '...' }) }),
);

// ✅ Typed and lint-clean
const [[updateArg]] = db.user.update.mock.calls as [[{ data: { metaData: string } }]];
expect(updateArg.data.metaData).toBe(JSON.stringify({ advertisingChannels: ['meta'] }));
```

## Structure

```
describe('ServiceName.methodName', () => {
  // happy paths first
  it('does the main thing', ...)
  it('handles edge case X', ...)

  // grouped error scenarios
  describe('when X fails', () => {
    beforeEach(() => { mock.mockRejectedValue(...) })
    it('re-throws the error', ...)
    it('does not mark as complete', ...)
  })
})
```

## Common Mistakes

**Asserting call counts:** `toHaveBeenCalledTimes(1)` breaks when behaviour is unchanged but implementation changes. Prefer `toHaveBeenCalledWith(...)` or just `toHaveBeenCalled()`.

**Testing logger calls:** Loggers are observability tools, not behavior. Skip them unless logging IS the feature under test.

**`toEqual` on partial objects:** Use `toMatchObject` when only some fields matter, `toContainEqual(expect.objectContaining(...))` for items in arrays.

**`as any` on private members:** Always use `as unknown as { property: Type }` — it's typed, self-documenting, and lint-clean.
