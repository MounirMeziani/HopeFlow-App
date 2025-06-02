# Database Planning Document: AI Well-being Navigator

## 1. Data Models

This section defines the core entities, their attributes, data types, relationships, and constraints, based on the AI Well-being Navigator PRD.

### 1.1. User

Represents an application user account.

| Attribute            | Data Type    | Constraints                           | Description                                                        |
| :------------------- | :----------- | :------------------------------------ | :----------------------------------------------------------------- |
| `user_id`          | UUID         | PRIMARY KEY, NOT NULL, UNIQUE         | Unique identifier for the user.                                    |
| `username`         | VARCHAR(255) | NOT NULL, UNIQUE                      | Pseudonymized username for community/display.                      |
| `hashed_password`  | VARCHAR(255) | NOT NULL                              | Securely hashed password.                                          |
| `email`            | VARCHAR(255) | UNIQUE, NULLABLE (Optional)           | User's email address (optional for core features).                 |
| `created_at`       | TIMESTAMP    | NOT NULL, DEFAULT CURRENT_TIMESTAMP   | Timestamp of user creation.                                        |
| `updated_at`       | TIMESTAMP    | NOT NULL, DEFAULT CURRENT_TIMESTAMP   | Timestamp of last update.                                          |
| `last_login_at`    | TIMESTAMP    | NULLABLE                              | Timestamp of last login.                                           |
| `is_premium`       | BOOLEAN      | NOT NULL, DEFAULT FALSE               | Indicates premium subscription status.                             |
| `is_moderator`     | BOOLEAN      | NOT NULL, DEFAULT FALSE               | Indicates moderator role for community.                            |
| `is_active`        | BOOLEAN      | NOT NULL, DEFAULT TRUE                | Account status (e.g., for soft deletion).                          |
| `demographic_data` | JSONB        | NULLABLE (Optional, requires consent) | Optional, anonymized demographic info (e.g., age range, location). |

Relationships:

- One `User` has many `WellBeingCheckin`s.
- One `User` has many `JournalEntry`s.
- One `User` has many `HealthyAIInteractionLog`s.
- One `User` has many `CommunityPost`s.
- One `User` has many `CommunityComment`s.
- One `User` has one `NotificationSetting`.
- One `User` has many `Report`s (reporting content/users).

### 1.2. WellBeingCheckin

Logs a user's self-reported anxiety level and triggers.

| Attribute         | Data Type | Constraints                                          | Description                                                                                                                                                                                      |
| :---------------- | :-------- | :--------------------------------------------------- | :----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `checkin_id`    | UUID      | PRIMARY KEY, NOT NULL, UNIQUE                        | Unique identifier for the check-in.                                                                                                                                                              |
| `user_id`       | UUID      | NOT NULL, FOREIGN KEY (User)                         | Link to the user who logged the check-in.                                                                                                                                                        |
| `checkin_time`  | TIMESTAMP | NOT NULL                                             | Timestamp of the check-in.                                                                                                                                                                       |
| `anxiety_level` | INTEGER   | NOT NULL, CHECK (`anxiety_level` BETWEEN 1 AND 10) | Self-reported anxiety level (1-10).                                                                                                                                                              |
| `trigger_ids`   | UUID[]    | NULLABLE                                             | Array of IDs linking to predefined triggers (if stored in a separate table, otherwise could be STRING[] or JSONB).*Assuming trigger IDs link to a future `Trigger` or `ContentItem` type.* |
| `notes`         | TEXT      | NULLABLE                                             | Optional brief notes about the check-in.                                                                                                                                                         |
| `created_at`    | TIMESTAMP | NOT NULL, DEFAULT CURRENT_TIMESTAMP                  | Timestamp record was created in DB.                                                                                                                                                              |

Relationships:

- Many `WellBeingCheckin`s belong to one `User`.

### 1.3. JournalEntry

Stores user's private journal entries. Sensitive data requires encryption.

| Attribute          | Data Type | Constraints                         | Description                                             |
| :----------------- | :-------- | :---------------------------------- | :------------------------------------------------------ |
| `journal_id`     | UUID      | PRIMARY KEY, NOT NULL, UNIQUE       | Unique identifier for the journal entry.                |
| `user_id`        | UUID      | NOT NULL, FOREIGN KEY (User)        | Link to the user.                                       |
| `entry_time`     | TIMESTAMP | NOT NULL                            | Timestamp the entry was made.                           |
| `encrypted_text` | TEXT      | NOT NULL                            | The encrypted journal content. Key management required. |
| `prompt_id`      | UUID      | NULLABLE, FOREIGN KEY (ContentItem) | Link to the prompt used, if any.                        |
| `created_at`     | TIMESTAMP | NOT NULL, DEFAULT CURRENT_TIMESTAMP | Timestamp record was created in DB.                     |
| `updated_at`     | TIMESTAMP | NOT NULL, DEFAULT CURRENT_TIMESTAMP | Timestamp of last update.                               |

Relationships:

- Many `JournalEntry`s belong to one `User`.
- One `JournalEntry` may optionally link to one `ContentItem` (if prompts are ContentItems).

### 1.4. HealthyAIInteractionLog

Records manual logs of user interaction with external AI tools.

| Attribute            | Data Type    | Constraints                         | Description                                                   |
| :------------------- | :----------- | :---------------------------------- | :------------------------------------------------------------ |
| `log_id`           | UUID         | PRIMARY KEY, NOT NULL, UNIQUE       | Unique identifier for the interaction log.                    |
| `user_id`          | UUID         | NOT NULL, FOREIGN KEY (User)        | Link to the user.                                             |
| `interaction_time` | TIMESTAMP    | NOT NULL                            | Timestamp of the interaction.                                 |
| `duration_minutes` | INTEGER      | NULLABLE                            | Estimated duration of interaction in minutes.                 |
| `tool_name`        | VARCHAR(255) | NULLABLE                            | Name or type of AI tool used (e.g., 'ChatGPT', 'Midjourney'). |
| `notes`            | TEXT         | NULLABLE                            | Optional notes about the interaction.                         |
| `created_at`       | TIMESTAMP    | NOT NULL, DEFAULT CURRENT_TIMESTAMP | Timestamp record was created in DB.                           |

Relationships:

- Many `HealthyAIInteractionLog`s belong to one `User`.

### 1.5. ContentCategory

Organizes educational/coping content.

| Attribute       | Data Type    | Constraints                         | Description                                                    |
| :-------------- | :----------- | :---------------------------------- | :------------------------------------------------------------- |
| `category_id` | UUID         | PRIMARY KEY, NOT NULL, UNIQUE       | Unique identifier for the category.                            |
| `name`        | VARCHAR(255) | NOT NULL, UNIQUE                    | Name of the category (e.g., 'AI Demystified', 'Coping Tools'). |
| `description` | TEXT         | NULLABLE                            | Description of the category.                                   |
| `order`       | INTEGER      | NOT NULL, DEFAULT 0                 | Display order for categories.                                  |
| `created_at`  | TIMESTAMP    | NOT NULL, DEFAULT CURRENT_TIMESTAMP | Timestamp record was created in DB.                            |
| `updated_at`  | TIMESTAMP    | NOT NULL, DEFAULT CURRENT_TIMESTAMP | Timestamp of last update.                                      |

Relationships:

- One `ContentCategory` has many `ContentItem`s.

### 1.6. ContentItem

Represents a piece of content (module, guide, article, tip, exercise, crisis resource, journal prompt).

| Attribute        | Data Type                                                                                                         | Constraints                             | Description                                                           |
| :--------------- | :---------------------------------------------------------------------------------------------------------------- | :-------------------------------------- | :-------------------------------------------------------------------- |
| `content_id`   | UUID                                                                                                              | PRIMARY KEY, NOT NULL, UNIQUE           | Unique identifier for the content item.                               |
| `category_id`  | UUID                                                                                                              | NULLABLE, FOREIGN KEY (ContentCategory) | Category the content belongs to. Crisis Resources might not have one. |
| `type`         | ENUM ('module', 'guide', 'article', 'tip', 'exercise_audio', 'exercise_cbt', 'crisis_resource', 'journal_prompt') | NOT NULL                                | Type of content.                                                      |
| `title`        | VARCHAR(255)                                                                                                      | NOT NULL                                | Title of the content.                                                 |
| `slug`         | VARCHAR(255)                                                                                                      | UNIQUE                                  | URL-friendly identifier.                                              |
| `summary`      | TEXT                                                                                                              | NULLABLE                                | Short summary or description.                                         |
| `full_content` | TEXT                                                                                                              | NULLABLE                                | Main body text (Markdown/HTML/structured JSON). Not for audio/crisis. |
| `audio_url`    | VARCHAR(512)                                                                                                      | NULLABLE                                | URL for audio content (e.g., meditations).                            |
| `image_url`    | VARCHAR(512)                                                                                                      | NULLABLE                                | URL for associated image.                                             |
| `is_premium`   | BOOLEAN                                                                                                           | NOT NULL, DEFAULT FALSE                 | Indicates if content requires premium access.                         |
| `is_published` | BOOLEAN                                                                                                           | NOT NULL, DEFAULT FALSE                 | Content publication status.                                           |
| `published_at` | TIMESTAMP                                                                                                         | NULLABLE                                | Date/time content was published.                                      |
| `created_at`   | TIMESTAMP                                                                                                         | NOT NULL, DEFAULT CURRENT_TIMESTAMP     | Timestamp record was created in DB.                                   |
| `updated_at`   | TIMESTAMP                                                                                                         | NOT NULL, DEFAULT CURRENT_TIMESTAMP     | Timestamp of last update.                                             |
| `order`        | INTEGER                                                                                                           | NOT NULL, DEFAULT 0                     | Display order within category/type.                                   |

Relationships:

- Many `ContentItem`s may belong to one `ContentCategory`.
- Many `JournalEntry`s may link to one `ContentItem` (if it's a prompt).
- Many `WellBeingCheckin`s may link to one `ContentItem` (if triggers are ContentItems).

### 1.7. CommunityCategory

Organizes community forum topics.

| Attribute       | Data Type    | Constraints                         | Description                                                             |
| :-------------- | :----------- | :---------------------------------- | :---------------------------------------------------------------------- |
| `category_id` | UUID         | PRIMARY KEY, NOT NULL, UNIQUE       | Unique identifier for the community category.                           |
| `name`        | VARCHAR(255) | NOT NULL, UNIQUE                    | Name of the category (e.g., 'General Discussion', 'Coping Strategies'). |
| `description` | TEXT         | NULLABLE                            | Description of the category.                                            |
| `order`       | INTEGER      | NOT NULL, DEFAULT 0                 | Display order for categories.                                           |
| `created_at`  | TIMESTAMP    | NOT NULL, DEFAULT CURRENT_TIMESTAMP | Timestamp record was created in DB.                                     |
| `updated_at`  | TIMESTAMP    | NOT NULL, DEFAULT CURRENT_TIMESTAMP | Timestamp of last update.                                               |

Relationships:

- One `CommunityCategory` has many `CommunityPost`s.

### 1.8. CommunityPost

Represents a forum post. User identity should be handled pseudonymously.

| Attribute          | Data Type    | Constraints                               | Description                                                      |
| :----------------- | :----------- | :---------------------------------------- | :--------------------------------------------------------------- |
| `post_id`        | UUID         | PRIMARY KEY, NOT NULL, UNIQUE             | Unique identifier for the post.                                  |
| `category_id`    | UUID         | NOT NULL, FOREIGN KEY (CommunityCategory) | Link to the category the post belongs to.                        |
| `author_user_id` | UUID         | NOT NULL, FOREIGN KEY (User)              | Link to the user who created the post (for backend/moderation).  |
| `pseudonym`      | VARCHAR(255) | NOT NULL                                  | Pseudonymized name displayed for the author (derived from User). |
| `title`          | VARCHAR(255) | NOT NULL                                  | Title of the post.                                               |
| `content`        | TEXT         | NOT NULL                                  | Body content of the post.                                        |
| `created_at`     | TIMESTAMP    | NOT NULL, DEFAULT CURRENT_TIMESTAMP       | Timestamp the post was created.                                  |
| `updated_at`     | TIMESTAMP    | NOT NULL, DEFAULT CURRENT_TIMESTAMP       | Timestamp of last update.                                        |
| `is_hidden`      | BOOLEAN      | NOT NULL, DEFAULT FALSE                   | Moderation status: hidden from public view.                      |
| `is_deleted`     | BOOLEAN      | NOT NULL, DEFAULT FALSE                   | Moderation status: soft deletion.                                |

Relationships:

- Many `CommunityPost`s belong to one `CommunityCategory`.
- Many `CommunityPost`s belong to one `User`.
- One `CommunityPost` has many `CommunityComment`s (Replies).
- One `CommunityPost` has many `Report`s.

### 1.9. CommunityComment (Reply)

Represents a reply to a community post. User identity handled pseudonymously.

| Attribute             | Data Type    | Constraints                              | Description                                      |
| :-------------------- | :----------- | :--------------------------------------- | :----------------------------------------------- |
| `comment_id`        | UUID         | PRIMARY KEY, NOT NULL, UNIQUE            | Unique identifier for the comment.               |
| `post_id`           | UUID         | NOT NULL, FOREIGN KEY (CommunityPost)    | Link to the post the comment replies to.         |
| `parent_comment_id` | UUID         | NULLABLE, FOREIGN KEY (CommunityComment) | Link to the parent comment for threaded replies. |
| `author_user_id`    | UUID         | NOT NULL, FOREIGN KEY (User)             | Link to the user who created the comment.        |
| `pseudonym`         | VARCHAR(255) | NOT NULL                                 | Pseudonymized name displayed for the author.     |
| `content`           | TEXT         | NOT NULL                                 | Body content of the comment.                     |
| `created_at`        | TIMESTAMP    | NOT NULL, DEFAULT CURRENT_TIMESTAMP      | Timestamp the comment was created.               |
| `updated_at`        | TIMESTAMP    | NOT NULL, DEFAULT CURRENT_TIMESTAMP      | Timestamp of last update.                        |
| `is_hidden`         | BOOLEAN      | NOT NULL, DEFAULT FALSE                  | Moderation status: hidden from public view.      |
| `is_deleted`        | BOOLEAN      | NOT NULL, DEFAULT FALSE                  | Moderation status: soft deletion.                |

Relationships:

- Many `CommunityComment`s belong to one `CommunityPost`.
- Many `CommunityComment`s belong to one `User`.
- A `CommunityComment` may have one optional `parent_comment_id` pointing to another `CommunityComment` for threading.
- One `CommunityComment` has many `Report`s.

### 1.10. Report

Records user reports against community content or users.

| Attribute               | Data Type                                    | Constraints                              | Description                              |
| :---------------------- | :------------------------------------------- | :--------------------------------------- | :--------------------------------------- |
| `report_id`           | UUID                                         | PRIMARY KEY, NOT NULL, UNIQUE            | Unique identifier for the report.        |
| `reporting_user_id`   | UUID                                         | NOT NULL, FOREIGN KEY (User)             | User who filed the report.               |
| `reported_post_id`    | UUID                                         | NULLABLE, FOREIGN KEY (CommunityPost)    | Link to the reported post, if any.       |
| `reported_comment_id` | UUID                                         | NULLABLE, FOREIGN KEY (CommunityComment) | Link to the reported comment, if any.    |
| `reported_user_id`    | UUID                                         | NULLABLE, FOREIGN KEY (User)             | Link to the reported user, if any.       |
| `reason`              | TEXT                                         | NOT NULL                                 | Description of the reason for reporting. |
| `created_at`          | TIMESTAMP                                    | NOT NULL, DEFAULT CURRENT_TIMESTAMP      | Timestamp the report was filed.          |
| `status`              | ENUM ('pending', 'under_review', 'resolved') | NOT NULL, DEFAULT 'pending'              | Moderation status of the report.         |
| `moderator_notes`     | TEXT                                         | NULLABLE                                 | Notes added by moderators during review. |

Constraints:

- CHECK constraint ensuring at least one of `reported_post_id`, `reported_comment_id`, or `reported_user_id` is NOT NULL.

Relationships:

- Many `Report`s belong to one `User` (reporting).
- Many `Report`s may target one `CommunityPost`.
- Many `Report`s may target one `CommunityComment`.
- Many `Report`s may target one `User` (reported).

### 1.11. NotificationSetting

Stores user preferences for notifications.

| Attribute                      | Data Type | Constraints                          | Description                                 |
| :----------------------------- | :-------- | :----------------------------------- | :------------------------------------------ |
| `setting_id`                 | UUID      | PRIMARY KEY, NOT NULL, UNIQUE        | Unique identifier for the setting record.   |
| `user_id`                    | UUID      | NOT NULL, UNIQUE, FOREIGN KEY (User) | Link to the user (one-to-one relationship). |
| `daily_tip_enabled`          | BOOLEAN   | NOT NULL, DEFAULT TRUE               | User opted into daily tips.                 |
| `community_activity_enabled` | BOOLEAN   | NOT NULL, DEFAULT TRUE               | User opted into community activity alerts.  |
| `updated_at`                 | TIMESTAMP | NOT NULL, DEFAULT CURRENT_TIMESTAMP  | Timestamp of last update.                   |

Relationships:

- One `NotificationSetting` belongs to one `User`.

### 1.12. AIInsight (Future)

*Planned future feature.* Stores generated insights based on user data (requires explicit consent).

| Attribute           | Data Type                                  | Constraints                   | Description                                                     |
| :------------------ | :----------------------------------------- | :---------------------------- | :-------------------------------------------------------------- |
| `insight_id`      | UUID                                       | PRIMARY KEY, NOT NULL, UNIQUE | Unique identifier for the insight.                              |
| `user_id`         | UUID                                       | NOT NULL, FOREIGN KEY (User)  | Link to the user.                                               |
| `generated_at`    | TIMESTAMP                                  | NOT NULL                      | Timestamp the insight was generated.                            |
| `insight_text`    | TEXT                                       | NOT NULL                      | The generated insight summary.                                  |
| `source_data_ids` | JSONB                                      | NULLABLE                      | IDs/references to source data (e.g., journal_ids, checkin_ids). |
| `data_type`       | ENUM ('journal', 'checkin', 'interaction') | NOT NULL                      | Type of data the insight is based on.                           |

Relationships:

- Many `AIInsight`s belong to one `User`.

*Note: CBT Exercise data structure could be simple (like JournalEntry, saving the final output text) or more complex (saving each step of the thought record). Starting simple with a TEXT field per entry linked to User, similar to JournalEntry, seems sufficient based on the PRD description.*

## 2. Database Schema (Conceptual)

This section outlines the structure of tables, keys, relationships, and indexing strategy.

Table List:

- `users`
- `well_being_checkins`
- `journal_entries`
- `healthy_ai_interaction_logs`
- `content_categories`
- `content_items`
- `community_categories`
- `community_posts`
- `community_comments`
- `reports`
- `notification_settings`
- `ai_insights` (Future)

Primary Keys:

Each table uses a `UUID` column as its primary key, ensuring global uniqueness and simplifying potential future distributed architectures.

- `users`: `user_id`
- `well_being_checkins`: `checkin_id`
- `journal_entries`: `journal_id`
- `healthy_ai_interaction_logs`: `log_id`
- `content_categories`: `category_id`
- `content_items`: `content_id`
- `community_categories`: `category_id`
- `community_posts`: `post_id`
- `community_comments`: `comment_id`
- `reports`: `report_id`
- `notification_settings`: `setting_id`
- `ai_insights`: `insight_id`

Foreign Keys and Relationships:

Foreign keys enforce referential integrity between related tables. Cascading rules are crucial for data consistency, especially upon user deletion.

| Table                           | Column(s)               | References Table         | References Column(s) | ON DELETE | ON UPDATE | Notes                                                        |
| :------------------------------ | :---------------------- | :----------------------- | :------------------- | :-------- | :-------- | :----------------------------------------------------------- |
| `well_being_checkins`         | `user_id`             | `users`                | `user_id`          | CASCADE   | CASCADE   | User's check-ins deleted if user is.                         |
| `journal_entries`             | `user_id`             | `users`                | `user_id`          | CASCADE   | CASCADE   | User's journal entries deleted if user.                      |
| `journal_entries`             | `prompt_id`           | `content_items`        | `content_id`       | SET NULL  | CASCADE   | Prompt link set to NULL if prompt deleted.                   |
| `healthy_ai_interaction_logs` | `user_id`             | `users`                | `user_id`          | CASCADE   | CASCADE   | Interaction logs deleted if user.                            |
| `content_items`               | `category_id`         | `content_categories`   | `category_id`      | SET NULL  | CASCADE   | Content category link set to NULL if category deleted.       |
| `community_posts`             | `category_id`         | `community_categories` | `category_id`      | RESTRICT  | CASCADE   | Cannot delete category if posts exist.                       |
| `community_posts`             | `author_user_id`      | `users`                | `user_id`          | SET NULL  | CASCADE   | Author link set to NULL if user deleted (pseudonym remains). |
| `community_comments`          | `post_id`             | `community_posts`      | `post_id`          | CASCADE   | CASCADE   | Comments deleted if parent post deleted.                     |
| `community_comments`          | `parent_comment_id`   | `community_comments`   | `comment_id`       | CASCADE   | CASCADE   | Child comments deleted if parent comment deleted.            |
| `community_comments`          | `author_user_id`      | `users`                | `user_id`          | SET NULL  | CASCADE   | Author link set to NULL if user deleted (pseudonym remains). |
| `reports`                     | `reporting_user_id`   | `users`                | `user_id`          | CASCADE   | CASCADE   | Reports filed by user deleted if user is.                    |
| `reports`                     | `reported_post_id`    | `community_posts`      | `post_id`          | CASCADE   | CASCADE   | Reports on a post deleted if post deleted.                   |
| `reports`                     | `reported_comment_id` | `community_comments`   | `comment_id`       | CASCADE   | CASCADE   | Reports on a comment deleted if comment deleted.             |
| `reports`                     | `reported_user_id`    | `users`                | `user_id`          | SET NULL  | CASCADE   | Reported user link set to NULL if user deleted.              |
| `notification_settings`       | `user_id`             | `users`                | `user_id`          | CASCADE   | CASCADE   | Settings deleted if user is.                                 |
| `ai_insights` (Future)        | `user_id`             | `users`                | `user_id`          | CASCADE   | CASCADE   | Insights deleted if user is.                                 |

*Note on User Deletion and Community Data:* The PRD states "Backend must cascade delete all related user data (check-ins, journaling, community posts - handled according to ethical policy regarding community data)." While check-ins and journaling are clearly private and can be fully cascaded, community posts/comments have a public aspect. The PRD implies pseudonymization for community data. Setting `author_user_id` to NULL on user deletion for community posts/comments while retaining the content and pseudonym ensures the thread integrity is kept while sensitive PII linkage is removed. The cascade delete on `reported_post_id`/`reported_comment_id` in `reports` is appropriate as the report is meaningless without the content it refers to.

Indexes:

Indexes are critical for query performance, especially on large tables.

- Foreign Key Columns: Automatically indexed by most RDBMS, but explicitly listing reinforces importance.
  - `well_being_checkins`: `user_id`
  - `journal_entries`: `user_id`, `prompt_id`
  - `healthy_ai_interaction_logs`: `user_id`
  - `content_items`: `category_id`
  - `community_posts`: `category_id`, `author_user_id`
  - `community_comments`: `post_id`, `parent_comment_id`, `author_user_id`
  - `reports`: `reporting_user_id`, `reported_post_id`, `reported_comment_id`, `reported_user_id`
  - `notification_settings`: `user_id`
  - `ai_insights`: `user_id`
- Frequently Queried/Filtered/Sorted Columns:
  - `users`: `username`, `email` (if used for login/recovery), `created_at`, `is_active`, `is_premium`, `is_moderator`
  - `well_being_checkins`: `checkin_time` (for history/trends), `anxiety_level` (for filtering/aggregating levels)
  - `journal_entries`: `entry_time` (for history)
  - `healthy_ai_interaction_logs`: `interaction_time` (for history/trends)
  - `content_items`: `type`, `slug`, `is_published`, `published_at`, `order`
  - `community_posts`: `created_at`, `is_hidden`, `is_deleted`
  - `community_comments`: `created_at`, `is_hidden`, `is_deleted`
  - `reports`: `created_at`, `status`
  - `ai_insights`: `generated_at`, `data_type`

Views:

While not strictly required initially, views could simplify complex queries anticipated by the Analytics Module.

- `user_wellbeing_summary_v`: A view potentially joining `users` and `well_being_checkins` to provide aggregate stats per user (e.g., average anxiety, check-in count).
- `community_thread_v`: A view combining `community_posts` and `community_comments` to retrieve entire discussion threads efficiently.

## 3. Database Technology Choice Rationale: PostgreSQL

Based on the PRD and application plan, PostgreSQL is the recommended database technology.

Justification:

1. Relational Integrity & ACID Compliance: The application involves highly structured data with clear relationships (Users, logs, content, community posts). Maintaining data consistency and integrity is paramount, especially for sensitive user data and transactional operations like user registration or saving a journal entry. PostgreSQL's strong adherence to ACID (Atomicity, Consistency, Isolation, Durability) properties guarantees reliable transactions.
2. Data Relationships: The schema is fundamentally relational (one user has many check-ins, one post has many comments). PostgreSQL excels at managing these complex relationships with robust foreign key constraints and joins.
3. JSONB Support: The PRD mentions potential `customFields` for user profiles or flexible storage for settings (`demographic_data`). PostgreSQL's native `JSONB` type provides efficient storage and querying of semi-structured data within the relational model, offering flexibility without resorting to a pure NoSQL solution for all data.
4. Scalability & Performance: PostgreSQL supports standard scaling techniques like read replicas for read-heavy workloads (common in content delivery, progress views, community browsing). Its indexing capabilities (including advanced index types like GIN for JSONB or full-text search if needed) ensure query performance remains acceptable as data volume grows. Connection pooling is well-supported to manage concurrent user connections efficiently.
5. Robust Security Features: PostgreSQL includes built-in features for access control (`GRANT`/`REVOKE` permissions), row-level security (RLS) for fine-grained data access based on user roles, and native support for SSL connections (for data in transit). These align directly with the PRD's mandate for strict access controls and secure handling.
6. Extensibility: PostgreSQL's support for extensions (e.g., `pgcrypto` for server-side encryption if required, although file-system/disk encryption is more common for encryption at rest) makes it adaptable to evolving needs.
7. Maturity & Community: As a mature, open-source database, PostgreSQL has extensive documentation, a large community, and wide support across hosting providers, easing development and deployment.

While a NoSQL database like MongoDB could offer schema flexibility for certain data types, the core of the application's data (user accounts, logs, relationships between community content) is inherently relational. A polyglot persistence approach *could* be considered (e.g., using MongoDB for logs or journal entries and PostgreSQL for users, content, community structure), but this adds operational complexity. PostgreSQL's `JSONB` capability provides a pragmatic balance, allowing flexibility where needed within a unified, strongly consistent database system that aligns with most of the application's data characteristics and security requirements.

## 4. Data Management & Security Strategies

Implementing robust data management and security is critical due to the sensitive nature of well-being and journaling data.

1. Data Encryption:

   * Encryption at Rest: All sensitive data (PII, `hashed_password`, `email`, `encrypted_text` in `journal_entries`, `demographic_data`, well-being logs) must be encrypted when stored on disk. This is typically achieved through full-disk encryption at the infrastructure level (e.g., using cloud provider features like AWS EBS encryption, Azure Disk Encryption, GCP Persistent Disk encryption) or database-level encryption (e.g., PostgreSQL's `pgcrypto` extension for specific columns, or TDE if using a managed service that provides it). AES-256 or equivalent is the required standard. Key management (KMS) is essential.
   * Encryption in Transit: All communication between the Frontend Clients, Backend API, and Database must be encrypted using strong cryptographic protocols. TLS 1.3+ is required for API calls and database connections to prevent eavesdropping and tampering.
2. Access Control:

   * Database Level: Implement Role-Based Access Control (RBAC) within PostgreSQL.
     * Define distinct database roles (e.g., `app_backend_role`, `moderator_admin_role`, `backup_role`).
     * Grant minimum necessary privileges (`SELECT`, `INSERT`, `UPDATE`, `DELETE`) on specific tables to each role (Principle of Least Privilege). The primary application backend role should have privileges required for core operations.
     * A dedicated moderator role should only have access to community tables (`community_categories`, `community_posts`, `community_comments`) and the `reports` table, possibly with restricted `UPDATE`/`DELETE` capabilities (e.g., soft delete flags).
     * Database user accounts used by applications/services must connect using specific roles, not a superuser account.
   * Application Level: The Backend API layer must enforce authentication (JWT validation) and authorization checks for every request, ensuring users can only access or modify data they are permitted to (e.g., a user can only fetch their own `WellBeingCheckin`s or `JournalEntry`s). Moderator actions must be restricted via RBAC check against the user's role obtained during authentication.
3. Backup and Recovery:

   * Implement a comprehensive backup strategy to ensure data durability and recovery from failures.
   * Regular Backups: Daily full backups and hourly incremental backups are a minimum requirement.
   * Point-in-Time Recovery (PITR): Configure WAL (Write-Ahead Logging) archiving to enable recovery to any specific moment in time, minimizing data loss.
   * Store backups securely, encrypted, and preferably off-site or in a separate cloud region.
   * Regularly test the backup and recovery process.
4. Data Retention & Archival:

   * Define data retention periods based on user agreements, privacy policies, and potential regulatory requirements.
   * Implement automated processes to identify and delete data that has passed its retention period (e.g., old interaction logs, or deleted user data after a grace period).
   * Consider archival strategies for older, less frequently accessed data. This could involve moving data to cheaper storage tiers within the database or exporting it to a data lake for long-term, potentially anonymized, storage.
5. Anonymization/Pseudonymization:

   * Community Data: As specified, community posts and comments link to users via `author_user_id` but display a `pseudonym`. Upon user deletion, the `author_user_id` is set to NULL, severing the direct PII link while preserving the pseudonym and content for thread continuity, respecting the community context.
   * Analytics/Research: For aggregated analytics or potential research purposes, data should be anonymized or pseudonymized such that individual users cannot be identified. This could involve removing direct identifiers (`user_id`), aggregating data points, adding noise, or using k-anonymity/differential privacy techniques, particularly for sensitive data like journal insights or granular logs. This process must adhere to user consent settings.

## 5. Scalability Considerations

The database design and technology choice inherently support several levels of scalability to accommodate growth:

1. Vertical Scaling: PostgreSQL can scale vertically by running on more powerful servers with more CPU, RAM, and faster storage. This is the simplest initial scaling step.
2. Read Replicas: For read-heavy workloads (serving content, fetching history, browsing community), read replicas can be deployed. Traffic is directed to replicas for reads, offloading the primary database, which handles writes. PostgreSQL's built-in replication capabilities are robust.
3. Connection Pooling: Using a connection pooler (like PgBouncer) is essential to manage a large number of client connections efficiently, preventing the database from being overwhelmed by connection overhead.
4. Indexing Strategy: The proposed comprehensive indexing strategy ensures that query performance remains acceptable as the volume of data grows by allowing the database to quickly locate relevant rows.
5. Schema Design: The normalized relational schema reduces data redundancy and potential inconsistencies, which is beneficial for write performance and data integrity at scale. Using UUIDs for primary keys avoids potential bottlenecks associated with sequential integer IDs in distributed environments.
6. Partitioning: For extremely large tables (e.g., `well_being_checkins`, `journal_entries`), PostgreSQL supports declarative partitioning (e.g., by date range), which can improve performance and manageability by dividing the data into smaller, more manageable chunks.
7. Sharding (Future): If the application scales to a point where vertical scaling and read replicas are insufficient, sharding (distributing data across multiple database instances) is a potential, albeit complex, future step. PostgreSQL can be sharded using extensions or external tools, though this requires significant architectural changes.

The chosen design provides a solid foundation for typical application growth, leveraging PostgreSQL's inherent strengths before requiring more complex distributed patterns.
