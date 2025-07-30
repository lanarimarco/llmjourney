# B£G15M Program Flow Documentation

## Program Overview
B£G15M is a Smeup program that manages definitions and containers for configuration data. The program handles data validation, scanning, and visualization based on different data sources (database vs. G75/G90 systems).

## Key Variable: $$OG15
The variable `$$OG15` is crucial to understanding the program's behavior:
- **$$OG15 = *OFF**: Database file B£G15M0L is available and can be used for caching
- **$$OG15 = *ON**: Database file B£G15M0L is not available, data must be read from G75/G90 sources

## Program Flow Description

### Initialization (£INIZI Subroutine)
1. Sets release information (`£DMSRL = 'V2R1'`)
2. Checks if database file `B£G15M0F` exists in QTEMP
3. If file doesn't exist, calls `B£WK20CL` to create it
4. Attempts to open database file `B£G15M0L`
5. Sets `$$OG15 = *IN35` (indicator 35 reflects the success/failure of the OPEN operation)

### Main Program Logic
The program accepts three main functions:
- **VER** (Verify): Validates a specific code
- **SCA** (Scan): Scans through available values
- **VIS** (Visualize): Shows selection list

### Data Loading Strategy Based on $$OG15

#### Scenario 1: $$OG15 = *OFF (Database Available)
When the database file is successfully opened:

**Container Reading (RLETC):**
- First tries to read container definitions from database using `CHAIN` and `READE` operations
- Loads container data directly from B£G15M0L file
- Faster access since data is cached in database

**Definition Reading (RLETD):**
- Attempts to read definitions from database first
- Uses keyed access (`KG15M0L CHAIN`) for efficient retrieval
- Loads complete definition records from BTLIBE field and related BTCD fields

#### Scenario 2: $$OG15 = *ON (Database Not Available)
When the database file cannot be opened:

**Container Reading (RLETC):**
- Falls back to G75 system calls
- Uses `£G75FU='RIT'` with `£G75ME='INI'` to initialize
- Reads from 'SCP_TAB' source
- **Does NOT write** to database (since `$$OG15 = *OFF` condition fails)
- Iterates through G75 data using `£G75ME='ALL'`

**Definition Reading (RLETD):**
- Falls back to G90 system calls
- Uses `£G90FU='LET'` with `£G90ME='SETLL'` 
- Reads from SCP_TAB members
- Processes data through `RLETDR` subroutine
- **Does NOT cache** results to database (since `$$OG15 = *OFF` condition fails)

### Performance Implications

#### $$OG15 = *OFF (Optimized Path)
- **Faster**: Direct database access
- **Cached**: Data is stored locally for subsequent calls
- **Efficient**: Keyed access reduces I/O operations
- **Persistent**: Data remains available across program calls

#### $$OG15 = *ON (Fallback Path)  
- **Slower**: Must call external G75/G90 systems each time
- **No Caching**: Data is not stored for reuse
- **Higher I/O**: Multiple system calls required
- **Real-time**: Always gets current data from source systems

### Function-Specific Behavior

#### VER (Verification) Function
1. Calls `RCHK` to prepare data arrays based on `$$OG15` status
2. Calls `RVER` to validate the requested code
3. If code starts with special characters (!?/), sets `£G1536 = *ON` and calls visualization
4. Otherwise searches through loaded array for matching code
5. Returns description if found, or sets error flag `£G1535 = *ON`

#### SCA (Scan) Function  
1. Calls `RCHK` to load data
2. Implements positioning logic based on `£G15ME` method
3. Returns next available code/description pair
4. Sets `£G1535 = *ON` if no more data available

#### VIS (Visualization) Function
1. Sets indicator 40 for display mode
2. Calls `GES2` subroutine to build selection array
3. Uses `£G08` (selection list utility) to display options
4. Returns selected code or sets error if cancelled

## Error Handling
- `£G1535 = *ON`: Code not found or end of data reached
- `£G1536 = *ON`: Special processing requested (codes starting with !?/)
- The program gracefully handles database unavailability by switching to G75/G90 sources

## Memory Management
- Uses arrays (SCH, SCH_EL, SCH_TX) with 6000 elements maximum
- Implements array copying for different contexts (SCH_CO, SCH_CE, SCH_CT)
- Clears arrays when type changes to prevent data contamination
