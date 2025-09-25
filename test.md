```

// =============================================================================

// JUNOS OS FUNDAMENTALS - JNCIA

// =============================================================================



// ACRONYMS \& ABBREVIATIONS

/\*

&nbsp;\* CLI - Command Line Interface

&nbsp;\* RE  - Routing Engine

&nbsp;\* PFE - Packet Forwarding Engine

&nbsp;\* RPD - Routing Protocol Daemon

&nbsp;\*/



// CONSTANTS AND DEFINITIONS

CONSTANTS:

&nbsp;   COMMIT\_TIMEOUT = 600              // Default for 'commit confirmed'

&nbsp;   PORT\_SSH = 22                     // Secure shell port

&nbsp;   PORT\_TELNET = 23                  // Telnet port

&nbsp;   ROLLBACK\_LIMIT = 50               // Maximum stored configurations



// -----------------------------------------------------------------------------

// I. CORE CONCEPTS

// -----------------------------------------------------------------------------



/\*

&nbsp;\* CONCEPT: Junos Architecture

&nbsp;\* PURPOSE: Clean separation between control and forwarding planes

&nbsp;\* KEY FACTS:

&nbsp;\*   - RE handles routing protocols and management

&nbsp;\*   - PFE handles packet forwarding at wire speed

&nbsp;\*   - Kernel mediates between planes

&nbsp;\*/



// -----------------------------------------------------------------------------

// II. PROCEDURES \& WORKFLOWS

// -----------------------------------------------------------------------------



PROCEDURE Configure\_Initial\_System()

BEGIN

&nbsp;   /\*

&nbsp;    \* CONCEPT: Initial System Configuration

&nbsp;    \* PURPOSE: Establish basic device identity and management access

&nbsp;    \* CLI HIERARCHY: \[edit system]

&nbsp;    \* KEY FACTS:

&nbsp;    \*   - Hostname required for meaningful identification

&nbsp;    \*   - Root password mandatory for configuration commits

&nbsp;    \*   - Time zone affects all log timestamps

&nbsp;    \*/

&nbsp;   

&nbsp;   // Config: set system host-name lab-device

&nbsp;   SET hostname = device\_name

&nbsp;   

&nbsp;   // Config: set system time-zone America/New\_York

&nbsp;   SET timezone = user\_timezone

&nbsp;   

&nbsp;   // Config: set system root-authentication encrypted-password "hash"

&nbsp;   SET root\_password = encrypted\_hash

&nbsp;   

&nbsp;   // Operation: commit

&nbsp;   COMMIT configuration

END



// -----------------------------------------------------------------------------

// III. MONITORING \& TROUBLESHOOTING

// -----------------------------------------------------------------------------



PROCEDURE Examine\_System\_Status()

BEGIN

&nbsp;   /\*

&nbsp;    \* CONCEPT: System Status Verification

&nbsp;    \* PURPOSE: Confirm system configuration is active and correct

&nbsp;    \*/

&nbsp;   

&nbsp;   // Verification: show system information

&nbsp;   system\_info = DISPLAY\_SYSTEM\_INFO()

&nbsp;   

&nbsp;   // --- EXPECTED OUTPUT SNIPPET ---

&nbsp;   // Hostname                 : lab-device

&nbsp;   // Model                    : srx300

&nbsp;   // Junos: 20.4R3.8

&nbsp;   // ^------------------^     ^----------^

&nbsp;   // |                        |

&nbsp;   // +-- Configured hostname  +-- Software version

&nbsp;   // --- END SNIPPET ---

&nbsp;   

&nbsp;   RETURN system\_info

END

```

