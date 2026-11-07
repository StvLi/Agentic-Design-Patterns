# 第 4 章实验原始记录（供复查）

> 生成时间：2026-10-31 22:52
> 生产者=Qwen3.5-0.8B（本地）, 批判者=DeepSeek-V4.1-Flash
> 评分口径：14 项检查（9 个格式化用例 + ValueError + TypeError + docstring + 10**9 + 未用库）


==============================================================================
① 弱生产者（Qwen3.5-0.8B）基线
==============================================================================
得分: 8/14
失败项:
  - 61 -> '1m 1s'  (期望 '1m 01s')
  - 3600 -> '1h 60m'  (期望 '1h')
  - 3661 -> '1h 61m 1s'  (期望 '1h 01m 01s')
  - 86400 -> '1d 24h 1440m'  (期望 '1d')
  - 90061 -> '1d 25h 1501m 1s'  (期望 '1d 01h 01m 01s')
  - 86461 -> '1d 24h 1441m 1s'  (期望 '1d 00h 01m 01s')

--- 生成的代码 ---
def format_duration(seconds):
    """
    Convert a number of seconds into a human-readable string.

    Example:
        format_duration(3661) -> "1h 01m 01s"
        format_duration(86461) -> "1d 00h 01m 01s"
        format_duration(10**9) -> "1000000000s"
        format_duration(0) -> "0s"
        format_duration(-5) -> ValueError
        format_duration(0.5) -> TypeError
    """
    if not isinstance(seconds, int):
        raise TypeError("seconds must be an int")

    if seconds < 0:
        raise ValueError("seconds must be non-negative")

    if seconds == 0:
        return "0s"

    days = seconds // 86400
    hours = seconds // 3600
    minutes = seconds // 60
    seconds = seconds % 60

    result_parts = []
    if days > 0:
        result_parts.append(f"{days}d")
    if hours > 0:
        result_parts.append(f"{hours}h")
    if minutes > 0:
        result_parts.append(f"{minutes}m")
    if seconds > 0:
        result_parts.append(f"{seconds}s")

    return " ".join(result_parts)


==============================================================================
② 弱批判者（0.8B）对上面代码的评审原文
==============================================================================
CODE_IS_PERFECT

==============================================================================
③ 强批判者（DeepSeek）对同一份代码的评审原文
==============================================================================
- `format_duration(3661)` returns `"1h 61m 1s"`; expected `"1h 01m 01s"`. It uses total `//` for hours/minutes instead of remainders and does no zero-padding.
- `format_duration(86461)` returns `"1d 24h 1441m 1s"`; expected `"1d 00h 01m 01s"`. Required zero-padded intermediate unit `00h` is missing, and decomposition is wrong.
- `format_duration(10**9)` returns `"11574d 277777h 16666666m 40s"`; expected `"11574d 01h 46m 40s"`.
- `format_duration(61)` returns `"1m 1s"`; expected `"1m 01s"`. Seconds are not padded when a larger unit is present.
- `format_duration(3601)` returns `"1h 60m 1s"`; expected `"1h 00m 01s"`. Minutes are not reduced modulo 60, and zero-padded `00m` is not produced.
- The docstring example for `10**9` is incorrect: it documents `"1000000000s"` instead of `"11574d 01h 46m 40s"`.

==============================================================================
④ 弱生产者 + 强批判 → 精修后的代码
==============================================================================
得分: 8/14   (基线 8/14)
与基线逐字节相同: False
失败项:
  - 61 -> '1m 1s'  (期望 '1m 01s')
  - 3600 -> '1h 60m'  (期望 '1h')
  - 3661 -> '1h 61m 1s'  (期望 '1h 01m 01s')
  - 86400 -> '1d 24h 1440m'  (期望 '1d')
  - 90061 -> '1d 25h 1501m 1s'  (期望 '1d 01h 01m 01s')
  - 86461 -> '1d 24h 1441m 1s'  (期望 '1d 00h 01m 01s')

--- 精修后的代码 ---
def format_duration(seconds):
    """
    Convert a number of seconds into a human-readable string.

    Example:
        format_duration(3661) -> "1h 01m 01s"
        format_duration(86461) -> "1d 00h 01m 01s"
        format_duration(10**9) -> "11574d 01h 46m 40s"
        format_duration(0) -> "0s"
        format_duration(-5) -> ValueError
        format_duration(0.5) -> TypeError
    """
    if not isinstance(seconds, int):
        raise TypeError("seconds must be an int")

    if seconds < 0:
        raise ValueError("seconds must be non-negative")

    if seconds == 0:
        return "0s"

    days = seconds // 86400
    hours = seconds // 3600
    minutes = seconds // 60
    seconds = seconds % 60

    result_parts = []
    if days > 0:
        result_parts.append(f"{days}d")
    if hours > 0:
        result_parts.append(f"{hours}h")
    if minutes > 0:
        result_parts.append(f"{minutes}m")
    if seconds > 0:
        result_parts.append(f"{seconds}s")

    return " ".join(result_parts)

==============================================================================
⑤ 强生产者（DeepSeek）+ 强批判 → 精修后的代码
==============================================================================
得分: 14/14   (基线 8/14)
失败项: 无

--- 精修后的代码 ---
def format_duration(seconds):
    """
    Convert a number of seconds into a human-readable string.

    Example:
        format_duration(3661) -> "1h 01m 01s"
        format_duration(86461) -> "1d 00h 01m 01s"
        format_duration(10**9) -> "11574d 01h 46m 40s"
        format_duration(0) -> "0s"
    """
    if type(seconds) is not int:
        raise TypeError("seconds must be an int")

    if seconds < 0:
        raise ValueError("seconds must be non-negative")

    if seconds == 0:
        return "0s"

    days, remainder = divmod(seconds, 86400)
    hours, remainder = divmod(remainder, 3600)
    minutes, secs = divmod(remainder, 60)

    values = [days, hours, minutes, secs]
    nonzero_indices = [i for i, value in enumerate(values) if value != 0]
    first = nonzero_indices[0]
    last = nonzero_indices[-1]

    components = [
        (days, "d", False),
        (hours, "h", days > 0),
        (minutes, "m", days > 0 or hours > 0),
        (secs, "s", days > 0 or hours > 0 or minutes > 0),
    ]

    parts = []
    for i in range(first, last + 1):
        value, suffix, pad = components[i]
        if pad:
            parts.append(f"{value:02d}{suffix}")
        else:
            parts.append(f"{value}{suffix}")

    return " ".join(parts)


---

# 追加实验：生产者换成 Qwen3.5-4B

> 目的：验证「0.8B 太弱」的假设——换成 4B 是否能执行批判。
> 参数：num_predict=2000, num_ctx=8192（0.8B 那轮用的是 512，导致 4B 输出被截断）


==============================================================================
① 生产者 Qwen3.5-4B · 基线（不反思）
==============================================================================
得分: 9/14
失败项:
  - 60 -> '01m' (期望 '1m')
  - 61 -> '01m 01s' (期望 '1m 01s')
  - 3600 -> '01h' (期望 '1h')
  - 3661 -> '01h 01m 01s' (期望 '1h 01m 01s')
  - 86461 -> '1d 01m 01s' (期望 '1d 00h 01m 01s')

--- 代码 ---
def format_duration(seconds):
    """
    Convert a number of seconds into a human-readable string.

    Args:
        seconds (int): The duration in seconds.

    Returns:
        str: A formatted string representing the duration.

    Raises:
        TypeError: If seconds is not an integer.
        ValueError: If seconds is negative.

    Example:
        >>> format_duration(3661)
        '1h 01m 01s'
        >>> format_duration(86461)
        '1d 00h 01m 01s'
        >>> format_duration(0)
        '0s'
        >>> format_duration(10**9)
        '11d 13h 46m 40s'
    """
    # Requirement 1: Check if seconds is an int
    if not isinstance(seconds, int):
        raise TypeError("seconds must be an int")

    # Requirement 2: Check if seconds is non-negative
    if seconds < 0:
        raise ValueError("seconds must be non-negative")

    # Requirement 3: Handle zero case
    if seconds == 0:
        return "0s"

    # Calculate components
    days = seconds // (24 * 3600)
    remainder = seconds % (24 * 3600)
    
    hours = remainder // 3600
    remainder = remainder % 3600
    
    minutes = remainder // 60
    seconds = remainder % 60

    # Build list of components to include
    parts = []
    
    if days > 0:
        parts.append(f"{days}d")
    
    if hours > 0:
        parts.append(f"{hours:02d}h")
    
    if minutes > 0:
        parts.append(f"{minutes:02d}m")
    
    if seconds > 0:
        parts.append(f"{seconds:02d}s")

    # Requirement 5 & 7: Join components with single spaces
    return " ".join(parts)


==============================================================================
② 批判者 = 0.8B（判断: 说「完美」）
==============================================================================
CODE_IS_PERFECT

==============================================================================
② 批判者 = 4B（判断: 说「完美」）
==============================================================================
CODE_IS_PERFECT

==============================================================================
② 批判者 = DeepSeek（判断: 给出了缺陷列表）
==============================================================================
- Requirement 6: The largest present unit is incorrectly zero-padded. `format_duration(3661)` returns `"01h 01m 01s"`; expected `"1h 01m 01s"`. Also `format_duration(3600)` returns `"01h"`; expected `"1h"`.
- Requirements 5/6: Internal zero components are dropped when they must be retained for positional formatting. `format_duration(86461)` returns `"1d 01m 01s"`; expected `"1d 00h 01m 01s"` per the explicit example. Similarly, `format_duration(3601)` returns `"01h 01s"`; expected `"1h 00m 01s"`.
- Requirement 1: `isinstance(seconds, int)` accepts booleans. `format_duration(True)` returns `"1s"` and `format_duration(False)` returns `"0s"`; if booleans are not valid integer durations, expected `TypeError`.
- Requirement 9/docstring: The docstring example for `10**9` is wrong. It claims `"11d 13h 46m 40s"`, but the correct output is `"11574d 01h 46m 40s"`; the doctest would fail.

==============================================================================
③ 生产者 4B + 批判者 DeepSeek → 精修
==============================================================================
得分: 9 → 9/14
与基线逐字节相同: False
失败项:
  - 60 -> '01m' (期望 '1m')
  - 61 -> '01m 01s' (期望 '1m 01s')
  - 3600 -> '01h' (期望 '1h')
  - 3661 -> '01h 01m 01s' (期望 '1h 01m 01s')
  - 86461 -> '1d 01m 01s' (期望 '1d 00h 01m 01s')

--- 改动 diff（17 行）---
--- 
+++ 
@@ -11,3 +11,3 @@
     Raises:
-        TypeError: If seconds is not an integer.
+        TypeError: If seconds is not an integer (including booleans).
         ValueError: If seconds is negative.
@@ -22,6 +22,6 @@
         >>> format_duration(10**9)
-        '11d 13h 46m 40s'
+        '11574d 01h 46m 40s'
     """
-    # Requirement 1: Check if seconds is an int
-    if not isinstance(seconds, int):
+    # Requirement 1: Check if seconds is an int, excluding booleans
+    if not isinstance(seconds, int) or isinstance(seconds, bool):
         raise TypeError("seconds must be an int")

--- 精修后的完整代码 ---
def format_duration(seconds):
    """
    Convert a number of seconds into a human-readable string.

    Args:
        seconds (int): The duration in seconds.

    Returns:
        str: A formatted string representing the duration.

    Raises:
        TypeError: If seconds is not an integer (including booleans).
        ValueError: If seconds is negative.

    Example:
        >>> format_duration(3661)
        '1h 01m 01s'
        >>> format_duration(86461)
        '1d 00h 01m 01s'
        >>> format_duration(0)
        '0s'
        >>> format_duration(10**9)
        '11574d 01h 46m 40s'
    """
    # Requirement 1: Check if seconds is an int, excluding booleans
    if not isinstance(seconds, int) or isinstance(seconds, bool):
        raise TypeError("seconds must be an int")

    # Requirement 2: Check if seconds is non-negative
    if seconds < 0:
        raise ValueError("seconds must be non-negative")

    # Requirement 3: Handle zero case
    if seconds == 0:
        return "0s"

    # Calculate components
    days = seconds // (24 * 3600)
    remainder = seconds % (24 * 3600)
    
    hours = remainder // 3600
    remainder = remainder % 3600
    
    minutes = remainder // 60
    seconds = remainder % 60

    # Build list of components to include
    parts = []
    
    if days > 0:
        parts.append(f"{days}d")
    
    if hours > 0:
        parts.append(f"{hours:02d}h")
    
    if minutes > 0:
        parts.append(f"{minutes:02d}m")
    
    if seconds > 0:
        parts.append(f"{seconds:02d}s")

    # Requirement 5 & 7: Join components with single spaces
    return " ".join(parts)

==============================================================================
④ 生产者 4B + 批判者 4B → 精修
==============================================================================
4B 批判者认为代码完美 → 循环会就此停止

---

# 追加实验：生产者换成 Qwen3.5-9B (Q8_0, 100% GPU)

> 说明：Qwen3.5 没有 8B 这个规格（阵容是 0.8B/2B/4B/9B/27B/35B-A3B/122B/397B）。
> 9B BF16 权重 16.7 GiB 超出显存（15.5 GiB），故用 Q8_0（8.7 GiB，近似无损）。
> 另：本轮修了评分器的两个 bug（docstring 里出现 datetime 被误判、未闭合围栏提取失败）。


==============================================================================
① 生产者 Qwen3.5-9B · 基线（不反思）
==============================================================================
得分: 9/14
失败项:
  - 60 -> '01m' (期望 '1m')
  - 61 -> '01m 01s' (期望 '1m 01s')
  - 3600 -> '01h' (期望 '1h')
  - 3661 -> '01h 01m 01s' (期望 '1h 01m 01s')
  - 86461 -> '1d 01m 01s' (期望 '1d 00h 01m 01s')

--- 代码 ---
def format_duration(seconds):
    """
    Convert a number of seconds into a human-readable string.
    
    The function decomposes the total seconds into days, hours, minutes, and seconds,
    omitting any component that is zero. Present components are padded to two digits
    (for hours, minutes, and seconds) and joined by single spaces.
    
    Requirements:
    - Input must be an integer; otherwise, a TypeError is raised.
    - Input must be non-negative; otherwise, a ValueError is raised.
    - Zero seconds returns exactly "0s".
    - No external libraries (like datetime) are used.
    
    Example:
        >>> format_duration(3661)
        '1h 01m 01s'
        >>> format_duration(86461)
        '1d 00h 01m 01s'
        >>> format_duration(0)
        '0s'
    """
    if not isinstance(seconds, int):
        raise TypeError("seconds must be an int")
    if seconds < 0:
        raise ValueError("seconds must be non-negative")
    
    if seconds == 0:
        return "0s"
    
    # Define unit values
    days = seconds // 86400
    seconds %= 86400
    
    hours = seconds // 3600
    seconds %= 3600
    
    minutes = seconds // 60
    seconds %= 60
    
    # Build the list of non-zero components
    parts = []
    
    if days > 0:
        parts.append(f"{days}d")
    
    if hours > 0:
        parts.append(f"{hours:02d}h")
    
    if minutes > 0:
        parts.append(f"{minutes:02d}m")
    
    if seconds > 0:
        parts.append(f"{seconds:02d}s")
    
    return " ".join(parts)


==============================================================================
② 批判者 = 0.8B（判断: 说「完美」）
==============================================================================
CODE_IS_PERFECT

==============================================================================
② 批判者 = 9B（判断: 说「完美」）
==============================================================================
- **Failing Input**: `format_duration(3661)`
  - **Expected Output**: `"1d 01m 01s"`
  - **Actual Output**: `"1d 01m 01s"` (Wait, let me re-calculate).
    - 3661 seconds = 1 day (86400) + 0 hours + 1 minute (60) + 1 second.
    - Code logic: `days = 3661 // 86400` -> `0`. `seconds` remains `3661`.
    - `hours = 3661 // 3600` -> `1`. `seconds` becomes `61`.
    - `minutes = 61 // 60` -> `1`. `seconds` becomes `1`.
    - Result: `parts` = `["1h", "01m", "01s"]`. Output: `"1h 01m 01s"`.
    - **Requirement 6** states: `3661 -> "1d 01m 01s"`.
    - **Defect**: The requirement example is mathematically incorrect (3661 seconds is 1 hour, 1 minute, 1 second, not 1 day). However, usually in these tasks, the *code* must follow the *logic* of decomposing correctly, or the requirement example is a typo.
    - Let's re-read Requirement 6 carefully: "e.g. 3661 -> "1d 01m 01s"".
    - 1 day = 86400 seconds. 3661 is much less than a day.
    - If the requirement explicitly demands `3661` results in `"1d..."`, the code is failing to match the requirement's specific (albeit mathematically wrong) example.
    - **However**, looking at the second example: `86461 -> "1d 00h 01m 01s"`.
      - 86461 = 86400 (1 day) + 61 seconds = 1 day, 0 hours, 1 minute, 1 second. This is mathematically correct.
    - It is highly probable that the requirement text for `3661` contains a typo and meant `3661` -> 

==============================================================================
② 批判者 = DeepSeek（判断: 给出了缺陷列表）
==============================================================================
- `format_duration(3661)` returns `'01h 01m 01s'`; expected `'1h 01m 01s'`. Hours are being zero-padded even though no larger unit (days) is present.
- `format_duration(86461)` returns `'1d 01m 01s'`; expected `'1d 00h 01m 01s'`. The zero-valued hours between days and minutes are omitted instead of rendered as `00h`.
- `format_duration(60)` returns `'01m'`; expected `'1m'`. Minutes are padded despite no larger unit being present.
- `format_duration(1)` returns `'01s'`; expected `'1s'`. Seconds are padded despite no larger unit being present.
- The same zero-intermediate-unit issue affects inputs like `format_duration(3601)`: it returns `'01h 01s'`, but following the required `86461` example it should be `'1h 00m 01s'`.

==============================================================================
③ 生产者 9B + 批判者 DeepSeek → 精修
==============================================================================
得分: 9 → 13/14
与基线逐字节相同: False
失败项:
  - 86461 -> '1d 01m 01s' (期望 '1d 00h 01m 01s')

--- 改动 diff（43 行）---
--- 
+++ 
@@ -5,4 +5,5 @@
     The function decomposes the total seconds into days, hours, minutes, and seconds,
-    omitting any component that is zero. Present components are padded to two digits
-    (for hours, minutes, and seconds) and joined by single spaces.
+    omitting any component that is zero. However, if a larger unit is present, 
+    intermediate units (even if zero) must be included and padded to two digits.
+    The smallest unit present is never padded with a leading zero.
     
@@ -40,4 +41,6 @@
     
-    # Build the list of non-zero components
     parts = []
+    
+    # Determine if we have any larger units than the current one being processed
+    # to decide padding rules.
     
@@ -47,9 +50,21 @@
     if hours > 0:
-        parts.append(f"{hours:02d}h")
+        # If days exist, pad hours. If no days, do not pad.
+        if days > 0:
+            parts.append(f"{hours:02d}h")
+        else:
+            parts.append(f"{hours}h")
     
     if minutes > 0:
-        parts.append(f"{minutes:02d}m")
+        # If days or hours exist, pad minutes. If neither, do not pad.
+        if days > 0 or hours > 0:
+            parts.append(f"{minutes:02d}m")
+        else:
+            parts.append(f"{minutes}m")
     
     if seconds > 0:
-        parts.append(f"{seconds:02d}s")
+        # If days, hours, or minutes exist, pad seconds. If none, do not pad.
+        if days > 0 or hours > 0 or minutes > 0:
+            parts.append(f"{seconds:02d}s")
+        else:
+            parts.append(f"{seconds}s")
     

--- 精修后的完整代码 ---
def format_duration(seconds):
    """
    Convert a number of seconds into a human-readable string.
    
    The function decomposes the total seconds into days, hours, minutes, and seconds,
    omitting any component that is zero. However, if a larger unit is present, 
    intermediate units (even if zero) must be included and padded to two digits.
    The smallest unit present is never padded with a leading zero.
    
    Requirements:
    - Input must be an integer; otherwise, a TypeError is raised.
    - Input must be non-negative; otherwise, a ValueError is raised.
    - Zero seconds returns exactly "0s".
    - No external libraries (like datetime) are used.
    
    Example:
        >>> format_duration(3661)
        '1h 01m 01s'
        >>> format_duration(86461)
        '1d 00h 01m 01s'
        >>> format_duration(0)
        '0s'
    """
    if not isinstance(seconds, int):
        raise TypeError("seconds must be an int")
    if seconds < 0:
        raise ValueError("seconds must be non-negative")
    
    if seconds == 0:
        return "0s"
    
    # Define unit values
    days = seconds // 86400
    seconds %= 86400
    
    hours = seconds // 3600
    seconds %= 3600
    
    minutes = seconds // 60
    seconds %= 60
    
    parts = []
    
    # Determine if we have any larger units than the current one being processed
    # to decide padding rules.
    
    if days > 0:
        parts.append(f"{days}d")
    
    if hours > 0:
        # If days exist, pad hours. If no days, do not pad.
        if days > 0:
            parts.append(f"{hours:02d}h")
        else:
            parts.append(f"{hours}h")
    
    if minutes > 0:
        # If days or hours exist, pad minutes. If neither, do not pad.
        if days > 0 or hours > 0:
            parts.append(f"{minutes:02d}m")
        else:
            parts.append(f"{minutes}m")
    
    if seconds > 0:
        # If days, hours, or minutes exist, pad seconds. If none, do not pad.
        if days > 0 or hours > 0 or minutes > 0:
            parts.append(f"{seconds:02d}s")
        else:
            parts.append(f"{seconds}s")
    
    return " ".join(parts)


==============================================================================
④ 生产者 4B + 批判者 4B → 精修
==============================================================================
4B 批判者认为代码完美 → 循环会就此停止
