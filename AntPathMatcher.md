#### relationship with spring
[跳转到spring初始化AntPathMatcher的位置](read.md#abstractapplicationcontext-constructor)  
AbstractApplicationContext 构造函数 getResourcePatternResolver() 获取到 PathMatchingResourcePatternResolver实例，antpathmatcher 是该实例的字段

#### AntPathMatcher
![](images/AntPathMatcher.png)


#### AntPathMatcher.doMatch
```java
protected boolean doMatch(String pattern, String path, boolean fullMatch,
    @Nullable Map<String, String> uriTemplateVariables) {

  if (path.startsWith(this.pathSeparator) != pattern.startsWith(this.pathSeparator)) {
    return false;
  }

  String[] pattDirs = tokenizePattern(pattern);
  if (fullMatch && this.caseSensitive && !isPotentialMatch(path, pattDirs)) {
    return false;
  }

  String[] pathDirs = tokenizePath(path);

  int pattIdxStart = 0;
  int pattIdxEnd = pattDirs.length - 1;
  int pathIdxStart = 0;
  int pathIdxEnd = pathDirs.length - 1;

  // Match all elements up to the first **
  while (pattIdxStart <= pattIdxEnd && pathIdxStart <= pathIdxEnd) {
    String pattDir = pattDirs[pattIdxStart];
    if ("**".equals(pattDir)) {
      break;
    }
    if (!matchStrings(pattDir, pathDirs[pathIdxStart], uriTemplateVariables)) {
      return false;
    }
    pattIdxStart++;
    pathIdxStart++;
  }

  if (pathIdxStart > pathIdxEnd) {
    // Path is exhausted, only match if rest of pattern is * or **'s
    if (pattIdxStart > pattIdxEnd) {
      return (pattern.endsWith(this.pathSeparator) == path.endsWith(this.pathSeparator));
    }
    if (!fullMatch) {
      return true;
    }
    if (pattIdxStart == pattIdxEnd && pattDirs[pattIdxStart].equals("*") && path.endsWith(this.pathSeparator)) {
      return true;
    }
    for (int i = pattIdxStart; i <= pattIdxEnd; i++) {
      if (!pattDirs[i].equals("**")) {
        return false;
      }
    }
    return true;
  }
  else if (pattIdxStart > pattIdxEnd) {
    // String not exhausted, but pattern is. Failure.
    return false;
  }
  else if (!fullMatch && "**".equals(pattDirs[pattIdxStart])) {
    // Path start definitely matches due to "**" part in pattern.
    return true;
  }

  // up to last '**'
  while (pattIdxStart <= pattIdxEnd && pathIdxStart <= pathIdxEnd) {
    String pattDir = pattDirs[pattIdxEnd];
    if (pattDir.equals("**")) {
      break;
    }
    if (!matchStrings(pattDir, pathDirs[pathIdxEnd], uriTemplateVariables)) {
      return false;
    }
    pattIdxEnd--;
    pathIdxEnd--;
  }
  if (pathIdxStart > pathIdxEnd) {
    // String is exhausted
    for (int i = pattIdxStart; i <= pattIdxEnd; i++) {
      if (!pattDirs[i].equals("**")) {
        return false;
      }
    }
    return true;
  }

  while (pattIdxStart != pattIdxEnd && pathIdxStart <= pathIdxEnd) {
    int patIdxTmp = -1;
    for (int i = pattIdxStart + 1; i <= pattIdxEnd; i++) {
      if (pattDirs[i].equals("**")) {
        patIdxTmp = i;
        break;
      }
    }
    if (patIdxTmp == pattIdxStart + 1) {
      // '**/**' situation, so skip one
      pattIdxStart++;
      continue;
    }
    // Find the pattern between padIdxStart & padIdxTmp in str between
    // strIdxStart & strIdxEnd
    int patLength = (patIdxTmp - pattIdxStart - 1);
    int strLength = (pathIdxEnd - pathIdxStart + 1);
    int foundIdx = -1;

    strLoop:
    for (int i = 0; i <= strLength - patLength; i++) {
      for (int j = 0; j < patLength; j++) {
        String subPat = pattDirs[pattIdxStart + j + 1];
        String subStr = pathDirs[pathIdxStart + i + j];
        if (!matchStrings(subPat, subStr, uriTemplateVariables)) {
          continue strLoop;
        }
      }
      foundIdx = pathIdxStart + i;
      break;
    }

    if (foundIdx == -1) {
      return false;
    }

    pattIdxStart = patIdxTmp;
    pathIdxStart = foundIdx + patLength;
  }

  for (int i = pattIdxStart; i <= pattIdxEnd; i++) {
    if (!pattDirs[i].equals("**")) {
      return false;
    }
  }

  return true;
}
```

#### String[] pattDirs = tokenizePattern(pattern);
```java
protected String[] tokenizePath(String path) {
  return StringUtils.tokenizeToStringArray(path, this.pathSeparator, this.trimTokens, true);
}
```

<details>    
<summary>StringUtils.tokenizeToStringArray</summary>

#### StringUtils.tokenizeToStringArray
```java
public static String[] tokenizeToStringArray(
		@Nullable String str, String delimiters, boolean trimTokens, boolean ignoreEmptyTokens) {

	if (str == null) {
		return new String[0];
	}

	StringTokenizer st = new StringTokenizer(str, delimiters);
	List<String> tokens = new ArrayList<>();
	while (st.hasMoreTokens()) {
		String token = st.nextToken();
		if (trimTokens) {
			token = token.trim();
		}
		if (!ignoreEmptyTokens || token.length() > 0) {
			tokens.add(token);
		}
	}
	return toStringArray(tokens);
}
```

#### StringTokenizer.hasMoreTokens
```java
// 设置newPosition为第一个非delimiters字符，判断从当前位置开始是否还有非delimiters字符
public boolean hasMoreTokens() {
    /*
     * Temporarily store this position and use it in the following
     * nextToken() method only if the delimiters haven't been changed in
     * that nextToken() invocation.
     */
    newPosition = skipDelimiters(currentPosition);
    return (newPosition < maxPosition);
}
```

#### newPosition = skipDelimiters(currentPosition);
```java
// 从startPos开始跳过所有的delimiter字符，返回第一个非delimiter字符
// 原则只要字符包含在delimiters中就跳过
private int skipDelimiters(int startPos) {
if (delimiters == null)
    throw new NullPointerException();

int position = startPos;
while (!retDelims && position < maxPosition) {
    if (!hasSurrogates) {
	char c = str.charAt(position);
	// 关键代码，当且仅当c，包含在delimiters中才会移动position
	// maxDelimCodePoint delimiters中最大字符的ascii码，这里采用短路或避免 indexof的性能损耗
	// 即：如果c都大于delimiters中最大的字符了，肯定不会包含在delimiters中，直接跳出循环
	if ((c > maxDelimCodePoint) || (delimiters.indexOf(c) < 0))
	    break;
	position++;
    } else {
	int c = str.codePointAt(position);
	if ((c > maxDelimCodePoint) || !isDelimiter(c)) {
	    break;
	}
	position += Character.charCount(c);
    }
}
return position;
}
```
#### StringUtils.nextToken
```java
public String nextToken() {
    /*
     * If next position already computed in hasMoreElements() and
     * delimiters have changed between the computation and this invocation,
     * then use the computed value.
     */

    currentPosition = (newPosition >= 0 && !delimsChanged) ?
        newPosition : skipDelimiters(currentPosition);

    /* Reset these anyway */
    delimsChanged = false;
    newPosition = -1;

    if (currentPosition >= maxPosition)
        throw new NoSuchElementException();
    int start = currentPosition;
    currentPosition = scanToken(currentPosition);
    // 这里start是第一个非delimiter位置，currentPosition是第一个delimiter位置或者最大位置
    // 返回值就是一个token
    return str.substring(start, currentPosition);
}
```

#### StringUtils.scanToken
```java
// 该方法的返回值，要么是第一个dilimiter位置，要么是最大位置
private int scanToken(int startPos) {
int position = startPos;
while (position < maxPosition) {
    if (!hasSurrogates) {
	char c = str.charAt(position);
	// 关键代码在第一个delimiter位置跳出
	if ((c <= maxDelimCodePoint) && (delimiters.indexOf(c) >= 0))
	    break;
	position++;
    } else {
	int c = str.codePointAt(position);
	if ((c <= maxDelimCodePoint) && isDelimiter(c))
	    break;
	position += Character.charCount(c);
    }
}
if (retDelims && (startPos == position)) {
    if (!hasSurrogates) {
	char c = str.charAt(position);
	if ((c <= maxDelimCodePoint) && (delimiters.indexOf(c) >= 0))
	    position++;
    } else {
	int c = str.codePointAt(position);
	if ((c <= maxDelimCodePoint) && isDelimiter(c))
	    position += Character.charCount(c);
    }
}
return position;
}
```
总结：StringUtils.tokenizeToStringArray 将一个字符串按照delimiters分隔为字符串数组
分隔逻辑为：先找到第一个不包含在delimiters中的字符，设置该字符的位置为起始位置a
再从a开始找到第一个包含在delimiters中的字符（或者字符串结尾）,设置改位置为b
那么[a,b)的字符串就是一个token

```
注意：
因为只是判断字符是否在delimiters中，没有考虑顺序，所以
StringUtils.tokenizeToStringArray("123ab321", "ba", false, true); 的返回值为 [123, 321]
即ab可以分隔ba，甚至可以分隔abbbabaababababa
123ab321 : 路径
ba ： 分隔符
false : 是否截取首尾空格
true ：是否保留空字符串
```
[回到 StringUtils.tokenizeToStringArray](#StringUtils.tokenizeToStringArray)
</details>
















