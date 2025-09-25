```ruby
// =============================================================================
// TOPIC: Secondary System Configuration - JNCIA
// =============================================================================

// ACRONYMS & ABBREVIATIONS
/*
 * RADIUS: Remote Authentication Dial-In User Service
 * TACACS+: Terminal Access Controller Access-Control System Plus
 * NTP: Network Time Protocol
 * SNMP: Simple Network Management Protocol
 * NMS: Network Management System
 * MIB: Management Information Base
 * OID: Object Identifier
 * FTP: File Transfer Protocol
 * SCP: Secure Copy Protocol
 */

// -----------------------------------------------------------------------------
// I. CORE CONCEPTS
// -----------------------------------------------------------------------------

/*
 * CONCEPT: Authentication vs. Authorization
 * PURPOSE: To distinguish between verifying a user's identity and defining the actions that user is permitted to perform.
 * OPERATION:
 * 1.  **Authentication**: This process answers the question, "Who are you?". It validates a user's credentials (e.g., username and password) against a database. Junos supports local, RADIUS, and TACACS+ databases.
 * 2.  **Authorization**: This process answers the question, "What are you allowed to do?". Once a user is authenticated, authorization determines which commands they can run and which parts of the configuration they can view or modify. This is managed through login classes and permissions.
 * KEY FACTS:
 * - **Authentication** is about IDENTITY.
 * - **Authorization** is about PERMISSIONS.
 * - A user must be successfully authenticated before authorization rules are applied.
 * - In Junos, all non-root users are subject to authorization; it cannot be disabled.
 */

/*
 * CONCEPT: System Logging (Syslog) vs. Tracing (Traceoptions)
 * PURPOSE: To provide two distinct methods for event recording: one for general system events and one for detailed, granular debugging.
 * OPERATION:
 * 1.  **System Logging (Syslog)**: Records high-level, system-wide events. This is used for general operational monitoring. Examples include user logins, interface status changes (up/down), and configuration commits. These are typically sent to the `/var/log/messages` file or a remote syslog server.
 * 2.  **Tracing (Traceoptions)**: The Junos equivalent of "debug." It records detailed, low-level operational information for a specific protocol or process, such as protocol state machine transitions or individual packet exchanges. Trace output is resource-intensive and is written to a dedicated file (e.g., `/var/log/ospf-trace`) to avoid cluttering the main system log.
 * KEY FACTS:
 * - Use **Syslog** for general, high-level monitoring.
 * - Use **Tracing** for deep, granular debugging of a specific process or protocol.
 * - Tracing should be enabled only when actively troubleshooting and disabled afterward to conserve system resources.
 */

/*
 * CONCEPT: SNMP Framework
 * PURPOSE: To provide a standardized protocol for monitoring and managing network devices.
 * OPERATION: The SNMP model consists of three main components:
 * 1.  **SNMP Manager**: Software running on a Network Management System (NMS) that polls for information and receives alerts.
 * 2.  **SNMP Agent**: Software running on the managed network device (the Junos device) that responds to manager queries and sends alerts.
 * 3.  **MIB (Management Information Base)**: A hierarchical database on the agent that stores operational data. The manager requests specific data points (OIDs) from the MIB.
 * The manager polls the agent for data, and the agent can send unsolicited alerts, called **traps**, to the manager to report significant events.
 * KEY FACTS:
 * - A Junos device acts as an **SNMP agent**.
 * - An **NMS** acts as the **SNMP manager**.
 * - **Polling** is when the manager requests data from the agent.
 * - **Traps** are when the agent sends an unsolicited alert to the manager.
 */

// -----------------------------------------------------------------------------
// II. PROCEDURES & WORKFLOWS
// -----------------------------------------------------------------------------

PROCEDURE Configure_User_Authentication(auth_order_list, remote_servers)
BEGIN
    /*
     * CONCEPT: User Authentication Methods
     * PURPOSE: To configure how the system validates user credentials, allowing for local or centralized authentication.
     * OPERATION: Junos checks authentication methods based on a configured order. For a login attempt, it tries the first method. If that server rejects the attempt, it tries the next. If a server is unreachable, it times out and tries the next. If all configured remote methods are unreachable, it will try the local password database as a final fallback.
     * IMPLEMENTATION: This procedure configures the authentication order and defines remote authentication servers.
     */

    // Config: set system authentication-order [ radius tacplus password ]
    Modify_Candidate_Configuration("set", "system authentication-order " + auth_order_list)

    FOR each server in remote_servers
        IF server.type == "radius" THEN
            // Config: set system radius-server <ip> secret <password>
            Modify_Candidate_Configuration("set", "system radius-server " + server.ip + " secret " + server.secret)
        ELSE IF server.type == "tacacs" THEN
            // Config: set system tacplus-server <ip> secret <password>
            Modify_Candidate_Configuration("set", "system tacplus-server " + server.ip + " secret " + server.secret)
        END IF
    END FOR
END

PROCEDURE Configure_User_Authorization(username, login_class, permissions_list)
BEGIN
    /*
     * CONCEPT: User Authorization
     * PURPOSE: To define custom access privileges for different classes of users.
     * IMPLEMENTATION: Creates a custom login class with specific permissions and assigns a user to that class.
     */

    // Config: set system login class <class_name> permissions [ <permission1> <permission2> ]
    Modify_Candidate_Configuration("set", "system login class " + login_class + " permissions " + permissions_list)

    // Example of allowing/denying specific commands/config hierarchies
    // Config: set system login class <class_name> allow-configuration "(interfaces)"
    // Config: set system login class <class_name> deny-commands "(file)"

    // Config: set system login user <username> class <class_name>
    Modify_Candidate_Configuration("set", "system login user " + username + " class " + login_class)

    // Config: set system login user <username> authentication plain-text-password
    Modify_Candidate_Configuration("set", "system login user " + username + " authentication plain-text-password")
END

PROCEDURE Configure_Protocol_Tracing(protocol, flags, file_options)
BEGIN
    /*
     * CONCEPT: Protocol Tracing
     * PURPOSE: To enable detailed debugging for a specific protocol and direct the output to a file.
     * IMPLEMENTATION: Configures `traceoptions` under a specific protocol hierarchy.
     */
    // Config: set protocols <protocol> traceoptions file <filename> size <size> files <num_files>
    Modify_Candidate_Configuration("set", "protocols " + protocol + " traceoptions file " + file_options.name)

    // Config: set protocols <protocol> traceoptions flag <flag> <detail_level>
    FOR each flag in flags
        Modify_Candidate_Configuration("set", "protocols " + protocol + " traceoptions flag " + flag)
    END FOR
END

PROCEDURE Configure_NTP_Client(boot_server_ip, server_ip_list)
BEGIN
    /*
     * CONCEPT: NTP Client Configuration
     * PURPOSE: To synchronize the device's system clock with a central time source.
     * IMPLEMENTATION: Defines one or more NTP servers for the device to peer with.
     * GOTCHAS: The `boot-server` is used once at boot time to set the initial time if it is drastically different from the NTP server. This prevents large time jumps that can disrupt NTP synchronization.
     */

    // Config: set system ntp boot-server <ip_address>
    Modify_Candidate_Configuration("set", "system ntp boot-server " + boot_server_ip)

    FOR each server_ip in server_ip_list
        // Config: set system ntp server <ip_address>
        Modify_Candidate_Configuration("set", "system ntp server " + server_ip)
    END FOR
END

PROCEDURE Configure_Automated_Backup(trigger, destination_url)
BEGIN
    /*
     * CONCEPT: Configuration Archiving
     * PURPOSE: To automatically back up the device configuration to a remote server for disaster recovery.
     * IMPLEMENTATION: Configures the system to transfer its configuration file to a remote FTP or SCP server.
     */
    // Config: set system archival configuration <trigger>
    Modify_Candidate_Configuration("set", "system archival configuration " + trigger) // trigger is 'transfer-on-commit' or 'transfer-interval <minutes>'

    // Config: set system archival configuration archive-sites "<url>" password "<password>"
    Modify_Candidate_Configuration("set", "system archival configuration archive-sites " + destination_url)
END

PROCEDURE Configure_SNMP_Agent(community_string, clients_subnet, trap_target_ip)
BEGIN
    /* CONCEPT: Basic SNMPv2c configuration. */
    // Config: set snmp description "My JUNOS Device"
    // Config: set snmp location "Data Center Rack 4"
    // Config: set snmp contact "Jim Davis - x1865"

    // Config: set snmp community <community_string> authorization read-only
    Modify_Candidate_Configuration("set", "snmp community " + community_string + " authorization read-only")

    // Config: set snmp community <community_string> clients <subnet>
    Modify_Candidate_Configuration("set", "snmp community " + community_string + " clients " + clients_subnet)

    // Config: set snmp trap-group <group_name> targets <ip_address>
    Modify_Candidate_Configuration("set", "snmp trap-group my-traps targets " + trap_target_ip)
END


// -----------------------------------------------------------------------------
// III. MONITORING & TROUBLESHOOTING
// -----------------------------------------------------------------------------

PROCEDURE Examine_Log_Files(filename, filter_expression)
BEGIN
    /* CONCEPT: Reading system log or trace files. */
    // Verification: show log <filename> | match <filter_expression>
    Display_File_Contents(filename, filter_expression)
END

PROCEDURE Monitor_Log_Files_Live(filename)
BEGIN
    /* CONCEPT: Viewing real-time updates to a log file. */
    // Operation: monitor start <filename>
    Display_Live_File_Updates(filename)

    // Operation: monitor stop
    // Halts the real-time monitoring.
END

PROCEDURE Verify_NTP_Synchronization()
BEGIN
    /*
     * CONCEPT: Verifying NTP Status
     * PURPOSE: To confirm that the device is successfully synchronized with an NTP peer.
     * IMPLEMENTATION: The `show ntp associations` command displays the status of all configured NTP peers.
     */
    // Verification: show ntp associations
    ntp_status = Get_NTP_Peers()
END

status = Verify_NTP_Synchronization()
// --- EXPECTED OUTPUT SNIPPET ---
//      remote           refid      st t when poll reach   delay   offset  jitter
// *10.210.14.173   10.210.8.73     4 u   63   64  377   0.268  -24.258   7.290
// ^--^             ^-----------^   ^-^
// |                |               |
// |                |               +-- Stratum of the peer server.
// |                |
// |                +-- The peer's synchronization source (reference ID).
// |
// +-- An asterisk indicates this is the currently selected peer for synchronization.
// --- END SNIPPET ---

PROCEDURE Verify_SNMP_Operation(mib_oid)
BEGIN
    /*
     * CONCEPT: Verifying SNMP MIB Access
     * PURPOSE: To confirm that the SNMP agent is running and that MIB data is accessible.
     * IMPLEMENTATION: The `show snmp mib walk` command simulates an NMS polling the local agent for all values under a specific OID.
     */
    // Verification: show snmp mib walk <oid_name_or_number>
    mib_data = Get_SNMP_MIB_Values(mib_oid)
END

data = Verify_SNMP_Operation("jnxOperatingDescr")
// --- EXPECTED OUTPUT SNIPPET ---
// jnxOperatingDescr.1.1.0.0 = midplane
// jnxOperatingDescr.2.1.1.0 = Power Supply 0
// jnxOperatingDescr.7.1.0.0 = FPC: EX3200-24T, 8 POE @ 0/*/*
// ^-----------------------^   ^---------------------------------^
// |                         |
// |                         +-- The value of the MIB object.
// |
// +-- The Object Identifier (OID) being queried.
// --- END SNIPPET ---
```
