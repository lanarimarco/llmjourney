# B£G15M Program: $$OG15 Scenarios Analysis

## Executive Summary
The B£G15M program operates in two distinct modes based on the status of the `$$OG15` variable, which is determined by the success or failure of opening the temporary database file `B£G15M0L`. This document provides a detailed comparison of both scenarios.

## Variable Definition
**$$OG15**: A 1-character indicator field that reflects the availability of the temporary database file
- Set to `*IN35` after attempting to open file `B£G15M0L`
- `*OFF` = Database file successfully opened and available
- `*ON` = Database file failed to open or is not available

---

## Scenario 1: $$OG15 = *OFF (Database Available Mode)

### Characteristics
- **Performance**: HIGH - Optimized database operations
- **Caching**: YES - Data is cached for subsequent calls
- **Data Source**: Local temporary database file
- **Reliability**: HIGH - Consistent, fast access

### Initialization Process
1. File `B£G15M0F` exists or is successfully created in QTEMP
2. `B£G15M0L` opens successfully (indicator 35 = OFF)
3. `$$OG15` is set to *OFF
4. Database is ready for cached operations

### Data Loading Workflow

#### Container Reading (RLETC Subroutine)
```
1. Check if containers already loaded ($C > 0)
   ├─ YES: Use cached data from SCH_CO arrays
   └─ NO: Load from database
2. Set BTTPK1 = 'CON'
3. CHAIN B£G15MR with key 'CON'
   ├─ Found: Read all container records using SETLL/READE
   │   └─ Load CON array from BTCD01+BTCD02
   └─ Not Found: Fall back to G75 system
4. For each container: Call RLETD to load definitions
```

#### Definition Reading (RLETD Subroutine)
```
1. Parse £G15TP (e.g., "CONTAINER.CHAPTER")
2. Set search key (BTTPK1, BTKEY1)
3. CHAIN B£G15MR with constructed key
   ├─ Found: Read definition records using SETLL/READE
   │   └─ Load SCH arrays from BTLIBE and BTCD01-BTCD11
   └─ Not Found: Fall back to G90 system
```

### Caching Behavior
- **Container Data**: Written to B£G15MR with BTTPK1='CON'
- **Definition Data**: Written to B£G15MR with BTTPK1='DEF'
- **Cache Key Structure**: BTTPK1 + BTKEY1 (container or type)
- **Cache Persistence**: Remains until QTEMP cleanup

### Performance Benefits
- **Fast Retrieval**: Direct keyed access via CHAIN/SETLL
- **Reduced I/O**: Eliminates repeated G75/G90 calls
- **Scalability**: Better performance with large datasets
- **Consistency**: Same data across multiple program calls

---

## Scenario 2: $$OG15 = *ON (G75/G90 Fallback Mode)

### Characteristics
- **Performance**: LOWER - External system calls required
- **Caching**: NO - Data retrieved fresh each time
- **Data Source**: G75/G90 external systems
- **Reliability**: MEDIUM - Dependent on external systems

### Initialization Process
1. File `B£G15M0F` cannot be created or accessed in QTEMP
2. `B£G15M0L` fails to open (indicator 35 = ON)
3. `$$OG15` is set to *ON
4. Program must use G75/G90 for all data access

### Data Loading Workflow

#### Container Reading (RLETC Subroutine)
```
1. Initialize G75 system call
2. Set £G75FU='RIT', £G75ME='INI'
3. Set £G75NF='SCP_TAB', £G75LC='S'
4. Execute £G75 call
5. Loop while *IN35 = *OFF:
   ├─ Load CON array from £G75MB + £G75DE
   ├─ NO caching ($$OG15 = *OFF condition fails)
   └─ Set £G75ME='ALL' for next iteration
6. For each container: Call RLETD to load definitions
```

#### Definition Reading (RLETD Subroutine)
```
1. Parse £G15TP to extract container and chapter
2. Initialize G90 system call
3. Set £G90FU='LET', £G90ME='SETLL'
4. Build £G90SO: 'Lib(*LIBL) Fil(SCP_TAB) Mem(container)'
5. Execute $G90 subroutine with replica 'K'
6. Loop through G90 data:
   ├─ Call RLETDR to process each record
   ├─ Load SCH arrays from £G90SO parsed values
   ├─ NO caching ($$OG15 = *OFF condition fails)
   └─ Continue until end of data or error
```

### No Caching Behavior
- **Critical Code Block**: `IF $$OG15 = *OFF` always fails
- **Impact**: WRITE statements to B£G15MR are never executed
- **Consequence**: Data must be retrieved fresh on every call
- **Memory Only**: Data exists only in program arrays during execution

### Performance Implications
- **Slower Response**: Each call requires G75/G90 system interaction
- **Network Dependency**: Performance varies with system load
- **Higher Resource Usage**: More CPU and I/O for repeated calls
- **Real-time Data**: Always current but at performance cost

---

## Comparative Analysis

| Aspect | $$OG15 = *OFF (Database) | $$OG15 = *ON (G75/G90) |
|--------|-------------------------|-------------------------|
| **Data Source** | B£G15M0L temporary file | G75/G90 external systems |
| **Access Method** | CHAIN/SETLL/READE | System program calls |
| **Caching** | ✅ Full caching enabled | ❌ No caching |
| **Performance** | 🟢 Fast (cached) | 🟡 Slower (real-time) |
| **Consistency** | 🟢 Same data per session | 🟡 Real-time updates |
| **Resource Usage** | 🟢 Low after initial load | 🔴 High on every call |
| **Failure Mode** | Falls back to G75/G90 | Direct G75/G90 calls |
| **Data Currency** | Cached until session end | Always current |

## Code Examples

### Database Mode ($$OG15 = *OFF)
```rpgle
C                   EVAL      *IN40=$$OG15          // *IN40 = *OFF
C  N40KG15M0L       CHAIN     B£G15MR               // CHAIN executes
C                   IF        *IN40 = *OFF          // Condition TRUE
C     KG15M0L       SETLL     B£G15MR               // Database read
C                   // ... load from database ...
C                   IF        $$OG15 = *OFF         // Condition TRUE
C                   WRITE     B£G15MR               // Caching occurs
C                   ENDIF
```

### G75/G90 Mode ($$OG15 = *ON)
```rpgle
C                   EVAL      *IN40=$$OG15          // *IN40 = *ON
C  N40KG15M0L       CHAIN     B£G15MR               // CHAIN skipped
C                   IF        *IN40 = *OFF          // Condition FALSE
C                   // ... database read skipped ...
C                   ELSE                            // Execute G75/G90
C                   EVAL      £G75FU='RIT'          // G75 system call
C                   // ... load from G75/G90 ...
C                   IF        $$OG15 = *OFF         // Condition FALSE
C                   // WRITE B£G15MR never executed // No caching
C                   ENDIF
```

## Troubleshooting Guide

### When $$OG15 = *ON (Unexpected)
**Possible Causes:**
1. QTEMP library authority issues
2. Insufficient disk space in QTEMP
3. B£WK20CL program not found or failed
4. File system errors during OPEN

**Diagnostic Steps:**
1. Check QTEMP library contents: `DSPLIB QTEMP`
2. Verify B£WK20CL availability: `CHKOBJ B£WK20CL *PGM`
3. Check system messages for file creation errors
4. Review job log for specific error messages

### Performance Issues
**$$OG15 = *OFF but slow:**
- Check database file size and organization
- Monitor for record locking issues
- Verify QTEMP space availability

**$$OG15 = *ON consistently:**
- Review system configuration for QTEMP access
- Check B£WK20CL program compilation
- Verify library list includes necessary components

## Best Practices

### For Development
1. **Monitor $$OG15 Status**: Log or display the value for debugging
2. **Test Both Modes**: Ensure application works in both scenarios
3. **Handle Gracefully**: Implement proper error handling for both paths

### For Production
1. **QTEMP Management**: Ensure adequate space and proper cleanup
2. **System Monitoring**: Watch for frequent G75/G90 calls indicating $$OG15=*ON
3. **Performance Tuning**: Optimize database file creation and access

### For Maintenance
1. **Documentation**: Always note which mode the system typically operates in
2. **Testing**: Verify both scenarios during system changes
3. **Monitoring**: Track performance differences between modes
