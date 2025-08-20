# Database Migration Planning Document

## Overview
This document outlines the database migration strategy for transitioning from airtime.com to cantina.com domains across all database systems.

---

## 🗄️ Database Systems Inventory

### Primary Databases
1. **PostgreSQL** - Main application database
2. **MongoDB** - Document store for media metadata
3. **Redis** - Cache and session storage
4. **DynamoDB** - AWS managed NoSQL (if applicable)
5. **RDS** - AWS managed relational databases

### Data Types Requiring Migration
- Configuration strings with domain references
- User webhook URLs
- OAuth redirect URIs
- Email templates with domain links
- API endpoint configurations
- Service discovery metadata
- CDN URLs in media records
- Notification callback URLs

---

## 📊 Database Audit Requirements

### Step 1: Identify Domain References
```sql
-- PostgreSQL: Find all airtime.com references
SELECT 
    table_name,
    column_name,
    COUNT(*) as occurrence_count
FROM (
    SELECT 
        table_name::text,
        column_name::text,
        row_to_json(t) as data
    FROM information_schema.columns c
    JOIN information_schema.tables t 
        ON c.table_name = t.table_name
    WHERE c.data_type IN ('text', 'varchar', 'json', 'jsonb')
) AS all_data
WHERE data::text LIKE '%airtime.com%'
GROUP BY table_name, column_name
ORDER BY occurrence_count DESC;
```

### Step 2: Common Tables to Check

#### Configuration Tables
```sql
-- configurations table
SELECT * FROM configurations 
WHERE value LIKE '%airtime.com%';

-- settings table  
SELECT * FROM settings
WHERE config_json::text LIKE '%airtime.com%';

-- system_config table
SELECT * FROM system_config
WHERE value LIKE '%airtime.com%' 
   OR config_data::text LIKE '%airtime.com%';
```

#### User Data Tables
```sql
-- users table - webhook URLs
SELECT id, email, webhook_url 
FROM users 
WHERE webhook_url LIKE '%airtime.com%';

-- oauth_applications table
SELECT * FROM oauth_applications
WHERE redirect_uri LIKE '%airtime.com%'
   OR callback_url LIKE '%airtime.com%';

-- api_keys table
SELECT * FROM api_keys
WHERE allowed_origins LIKE '%airtime.com%';
```

#### Media & Content Tables
```sql
-- media_assets table
SELECT * FROM media_assets
WHERE cdn_url LIKE '%airtime.com%'
   OR thumbnail_url LIKE '%airtime.com%';

-- email_templates table
SELECT * FROM email_templates
WHERE body LIKE '%airtime.com%'
   OR subject LIKE '%airtime.com%';
```

---

## 🔄 Migration Strategy

### Phase-Based Approach

#### Phase 1: Read-Only Audit (Day 1-2)
1. Run audit queries on all databases
2. Document all tables/columns with airtime.com
3. Count affected records
4. Estimate migration time
5. Identify high-risk data

#### Phase 2: Backup & Preparation (Day 3)
```bash
# PostgreSQL backup
pg_dump -h prod-db.airtime.com -U admin -d maindb > maindb_backup_$(date +%Y%m%d).sql

# MongoDB backup
mongodump --uri="mongodb://prod-mongo.airtime.com:27017" --out=mongo_backup_$(date +%Y%m%d)

# Redis snapshot
redis-cli -h prod-redis.airtime.com BGSAVE
```

#### Phase 3: Staging Migration Test (Day 4-5)
```sql
-- Create migration procedures
CREATE OR REPLACE FUNCTION migrate_domain_references()
RETURNS void AS $$
BEGIN
    -- Start transaction
    BEGIN;
    
    -- Update configurations
    UPDATE configurations 
    SET value = REPLACE(value, 'airtime.com', 'cantina.com'),
        updated_at = NOW(),
        updated_by = 'migration_script'
    WHERE value LIKE '%airtime.com%';
    
    -- Log changes
    INSERT INTO migration_log (table_name, records_updated, timestamp)
    VALUES ('configurations', ROW_COUNT(), NOW());
    
    -- Commit if successful
    COMMIT;
EXCEPTION
    WHEN OTHERS THEN
        -- Rollback on error
        ROLLBACK;
        RAISE;
END;
$$ LANGUAGE plpgsql;
```

#### Phase 4: Production Migration (Day 6)

---

## 🛠️ Migration Scripts

### PostgreSQL Migration Script
```sql
-- Main migration script
DO $$
DECLARE
    v_start_time timestamp;
    v_end_time timestamp;
    v_records_updated integer;
BEGIN
    v_start_time := clock_timestamp();
    
    -- Create backup table
    CREATE TABLE IF NOT EXISTS migration_backup_20250820 AS
    SELECT * FROM configurations WHERE value LIKE '%airtime.com%';
    
    -- Update configurations
    UPDATE configurations 
    SET value = REPLACE(value, 'airtime.com', 'cantina.com')
    WHERE value LIKE '%airtime.com%';
    GET DIAGNOSTICS v_records_updated = ROW_COUNT;
    RAISE NOTICE 'Updated % records in configurations', v_records_updated;
    
    -- Update user webhooks
    UPDATE users 
    SET webhook_url = REPLACE(webhook_url, 'airtime.com', 'cantina.com')
    WHERE webhook_url LIKE '%airtime.com%';
    GET DIAGNOSTICS v_records_updated = ROW_COUNT;
    RAISE NOTICE 'Updated % user webhook URLs', v_records_updated;
    
    -- Update JSON fields
    UPDATE settings
    SET config_json = 
        jsonb_set(
            config_json,
            '{}',
            replace(config_json::text, 'airtime.com', 'cantina.com')::jsonb
        )
    WHERE config_json::text LIKE '%airtime.com%';
    GET DIAGNOSTICS v_records_updated = ROW_COUNT;
    RAISE NOTICE 'Updated % JSON configurations', v_records_updated;
    
    v_end_time := clock_timestamp();
    RAISE NOTICE 'Migration completed in %', v_end_time - v_start_time;
END $$;
```

### MongoDB Migration Script
```javascript
// MongoDB migration script
db.getCollectionNames().forEach(function(collName) {
    var count = 0;
    
    // Search for airtime.com in all documents
    db[collName].find({
        $or: [
            {url: /airtime\.com/},
            {endpoint: /airtime\.com/},
            {webhook: /airtime\.com/},
            {config: /airtime\.com/}
        ]
    }).forEach(function(doc) {
        // Deep replace in document
        var updated = JSON.parse(
            JSON.stringify(doc).replace(/airtime\.com/g, 'cantina.com')
        );
        
        // Update document
        db[collName].replaceOne({_id: doc._id}, updated);
        count++;
    });
    
    if (count > 0) {
        print("Updated " + count + " documents in " + collName);
    }
});
```

### Redis Migration Script
```python
#!/usr/bin/env python3
import redis
import json
import re

def migrate_redis_keys():
    r = redis.Redis(host='prod-redis.airtime.com', port=6379, decode_responses=True)
    
    migrated_count = 0
    pattern = re.compile(r'airtime\.com')
    
    # Scan all keys
    for key in r.scan_iter("*"):
        key_type = r.type(key)
        
        if key_type == 'string':
            value = r.get(key)
            if value and 'airtime.com' in value:
                new_value = value.replace('airtime.com', 'cantina.com')
                r.set(key, new_value)
                migrated_count += 1
                print(f"Updated key: {key}")
        
        elif key_type == 'hash':
            hash_data = r.hgetall(key)
            for field, value in hash_data.items():
                if 'airtime.com' in value:
                    new_value = value.replace('airtime.com', 'cantina.com')
                    r.hset(key, field, new_value)
                    migrated_count += 1
                    print(f"Updated hash field: {key}:{field}")
    
    print(f"Total keys migrated: {migrated_count}")

if __name__ == "__main__":
    migrate_redis_keys()
```

---

## 🔍 Validation Queries

### Pre-Migration Validation
```sql
-- Count all occurrences before migration
CREATE TABLE migration_pre_check AS
SELECT 
    'configurations' as table_name,
    COUNT(*) as airtime_count
FROM configurations
WHERE value LIKE '%airtime.com%'
UNION ALL
SELECT 
    'users' as table_name,
    COUNT(*) as airtime_count
FROM users
WHERE webhook_url LIKE '%airtime.com%'
UNION ALL
SELECT 
    'settings' as table_name,
    COUNT(*) as airtime_count
FROM settings
WHERE config_json::text LIKE '%airtime.com%';
```

### Post-Migration Validation
```sql
-- Verify no airtime.com remains
SELECT 
    'configurations' as table_name,
    COUNT(*) as remaining_count
FROM configurations
WHERE value LIKE '%airtime.com%'
UNION ALL
SELECT 
    'users' as table_name,
    COUNT(*) as remaining_count
FROM users
WHERE webhook_url LIKE '%airtime.com%'
UNION ALL
SELECT 
    'settings' as table_name,
    COUNT(*) as remaining_count
FROM settings
WHERE config_json::text LIKE '%airtime.com%';

-- All counts should be 0
```

---

## 🔄 Rollback Plan

### PostgreSQL Rollback
```sql
-- Restore from backup table
BEGIN;

-- Restore configurations
UPDATE configurations c
SET value = b.value
FROM migration_backup_20250820 b
WHERE c.id = b.id;

-- Restore users
UPDATE users u
SET webhook_url = b.webhook_url  
FROM users_backup_20250820 b
WHERE u.id = b.id;

COMMIT;
```

### MongoDB Rollback
```javascript
// Restore from backup
mongorestore --uri="mongodb://prod-mongo.cantina.com:27017" \
    --dir=mongo_backup_20250820 \
    --drop
```

---

## ⚠️ Risk Assessment

### High Risk Areas
1. **JSON/JSONB fields** - Complex nested structures
2. **Encrypted fields** - May not be searchable
3. **Binary data** - Embedded URLs in blobs
4. **Cache invalidation** - Redis keys may be cached in app
5. **Foreign systems** - Third-party webhooks

### Mitigation Strategies
1. Run in maintenance window
2. Use database transactions where possible
3. Create comprehensive backups
4. Test rollback procedures
5. Monitor application logs during migration
6. Have database team on standby

---

## 📋 Migration Checklist

### Pre-Migration
- [ ] Audit all databases for airtime.com references
- [ ] Create full database backups
- [ ] Test migration scripts in staging
- [ ] Prepare rollback scripts
- [ ] Schedule maintenance window
- [ ] Notify stakeholders

### During Migration
- [ ] Set applications to maintenance mode
- [ ] Run backup verification
- [ ] Execute migration scripts
- [ ] Run validation queries
- [ ] Test critical flows
- [ ] Monitor error logs

### Post-Migration
- [ ] Remove maintenance mode
- [ ] Monitor application performance
- [ ] Verify webhook deliveries
- [ ] Check third-party integrations
- [ ] Archive backup data
- [ ] Document any issues

---

## 📊 Estimated Impact

| Database | Tables | Records | Migration Time |
|----------|--------|---------|---------------|
| PostgreSQL | 15-20 | ~50,000 | 30 minutes |
| MongoDB | 10-15 | ~100,000 | 45 minutes |
| Redis | N/A | ~5,000 keys | 15 minutes |
| **Total** | **30-40** | **~155,000** | **90 minutes** |

---

## 🚨 Emergency Contacts

- **Database Team Lead**: [Contact Info]
- **DBA On-Call**: [Contact Info]
- **Application Team**: [Contact Info]
- **DevOps**: [Contact Info]

---

## 📝 Notes

1. **Test in staging first** - Never run untested migrations in production
2. **Use transactions** - Ensure atomicity where possible
3. **Monitor replication** - Check replica lag during migration
4. **Cache warming** - May need to warm caches after migration
5. **Index rebuilding** - Large updates may require index maintenance

**Migration Window**: 2-3 hours (including validation)  
**Rollback Time**: 30 minutes maximum  
**Risk Level**: MEDIUM-HIGH (data modification)