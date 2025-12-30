run: go run main.go

- database:
```typescript
CREATE TYPE link_status AS ENUM ('pending', 'started');
CREATE TYPE link_type AS ENUM ('die', 'undefined', 'public', 'private');

CREATE TYPE token_status AS ENUM ('active', 'inactive', 'limit', 'die');
CREATE TYPE proxy_status AS ENUM ('active', 'inactive');

CREATE TYPE vps_status AS ENUM ('live', 'die');

CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    username VARCHAR(255) NOT NULL,
    password VARCHAR(255) NOT NULL,
    level INT DEFAULT 0,
    is_deleted BOOLEAN DEFAULT FALSE,
    created_at TIMESTAMP(6) DEFAULT (CURRENT_TIMESTAMP AT TIME ZONE 'UTC') NOT NULL
);

CREATE TABLE links (
    id SERIAL PRIMARY KEY,
    user_id INT NOT NULL,
    link_name VARCHAR(255),
    content TEXT,
    link_url VARCHAR(255) NOT NULL,
    post_id VARCHAR(255),
    post_id_v1 VARCHAR(255),
    page_id VARCHAR(255),
    last_comment_time TIMESTAMP(6) DEFAULT (CURRENT_TIMESTAMP AT TIME ZONE 'UTC') NOT NULL,
    delay_time INT DEFAULT 0,
    status link_status NOT NULL DEFAULT 'pending',
    type link_type NOT NULL DEFAULT 'undefined',
    process BOOLEAN DEFAULT FALSE,
    created_at TIMESTAMP(6) DEFAULT (CURRENT_TIMESTAMP AT TIME ZONE 'UTC') NOT NULL,
    is_deleted BOOLEAN DEFAULT FALSE,
    CONSTRAINT fk_links_user
        FOREIGN KEY (user_id)
        REFERENCES users(id)
);

CREATE TABLE token (
    id SERIAL PRIMARY KEY,
    token_value VARCHAR(255) NOT NULL,
    token_value_v1 VARCHAR(255),
    status token_status NOT NULL DEFAULT 'active',
    type SMALLINT NOT NULL DEFAULT 1
);

CREATE TABLE proxy (
    id SERIAL PRIMARY KEY,
    proxy_address VARCHAR(100) NOT NULL,
    status proxy_status NOT NULL DEFAULT 'active',
    is_fb_block BOOLEAN NOT NULL DEFAULT FALSE
);

CREATE TABLE cookie (
    id SERIAL PRIMARY KEY,
    cookie TEXT NOT NULL,
    created_by INT NOT NULL,
    token VARCHAR(255),
    status token_status NOT NULL DEFAULT 'active',
    CONSTRAINT fk_cookie_user
        FOREIGN KEY (created_by)
        REFERENCES users(id)
);


CREATE TABLE comments (
    id SERIAL PRIMARY KEY,
    post_id VARCHAR(255) NOT NULL,
    user_uid VARCHAR(255),
    username VARCHAR(255),
    message TEXT,
    created_at TIMESTAMP(6) DEFAULT (CURRENT_TIMESTAMP AT TIME ZONE 'UTC') NOT NULL,
    phone_number VARCHAR(255),
    cmt_id VARCHAR(255) NOT NULL,
    link_id INT NOT NULL DEFAULT 0,
    hide_cmt BOOLEAN NOT NULL DEFAULT FALSE,
    CONSTRAINT fk_comments_link
        FOREIGN KEY (link_id)
        REFERENCES links(id)
);

CREATE TABLE vps (
    id SERIAL PRIMARY KEY,
    ip VARCHAR(255) NOT NULL,
    port INT NOT NULL,
    speed VARCHAR(255) NOT NULL DEFAULT '0',
    status vps_status NOT NULL DEFAULT 'live'
);

CREATE TABLE orders (
    id BIGSERIAL PRIMARY KEY,
    user_id BIGINT NOT NULL,
    content TEXT,
    amount NUMERIC(20, 2) NOT NULL DEFAULT 0.00,
    status VARCHAR(20) NOT NULL DEFAULT 'pending' CHECK (status IN ('pending', 'confirmed', 'rejected', 'cancelled')),
    created_at TIMESTAMP(6) DEFAULT (CURRENT_TIMESTAMP AT TIME ZONE 'UTC') NOT NULL,
    
    CONSTRAINT fk_user FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE
);

-- Tạo indexes
CREATE INDEX idx_orders_user_id ON orders(user_id);
CREATE INDEX idx_orders_status ON orders(status);
CREATE INDEX idx_orders_created_at ON orders(created_at);


CREATE INDEX idx_links_user_id ON links(user_id);
CREATE INDEX idx_comments_link_id ON comments(link_id);
CREATE INDEX idx_links_status ON links(status);
CREATE INDEX idx_links_type ON links(type);


ALTER TABLE comments
ADD CONSTRAINT uq_comments_link_cmt
UNIQUE (link_id, cmt_id);

ALTER TABLE links
ADD CONSTRAINT uq_links_user_post
UNIQUE (user_id, post_id);


```