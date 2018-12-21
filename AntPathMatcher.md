#### relationship with spring
![relationship with spring](read.md#abstractapplicationcontext-constructor)
AbstractApplicationContext 构造函数 getResourcePatternResolver() 获取到 PathMatchingResourcePatternResolver实例，antpathmatcher 是该实例的字段

#### AntPathMatcher
[](images/AntPathMatcher.png)


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

#### StringUtils.hasMoreTokens
```java
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
    return str.substring(start, currentPosition);
}
```

















