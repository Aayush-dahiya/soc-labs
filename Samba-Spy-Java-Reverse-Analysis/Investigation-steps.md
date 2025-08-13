# Investigation Steps - Samba Spy Java Code Review

## Objective
Analyze a suspicious Java program provided in a 7z archive to understand its logic and extract key Indicators of Compromise (IOCs).

## Tools Used
- 7-Zip (to extract files)
- JD-GUI (Java Decompiler)
- LetsDefend Lab Environment

## Process

1. **File Extraction**
   - Unzipped the provided `challenge.7z` archive.
   - Located a `1.jar` file inside.

2. **Decompilation**
   - Opened the JAR in JD-GUI to view its source code.

3. **Code Review**
   - Found method `isRunningInVM` responsible for anti-VM checks.
   - Found that execution only continues if system language = `Italian`.
   - Located `extractLibs` method for unpacking `Prodotto.zip` to `C:\Users\Public\`.

4. **Payload Execution Logic**
   - Program searches for `Prodotto.png` (actually a JAR file).
   - Executes via:
     ```
     java -jar C:\Users\Public\Prodotto.png
     ```

5. **Additional Anti-VM Check**
   - Runs:
     ```
     wmic baseboard get manufacturer
     ```
   - Compares against a list of four known VM vendor names. 

## Outcome
Documented program behavior and extracted IOCs for possible SOC detection/alerting.

