---
title: EXISTS
---
Early Releases: Single-key request only. Requests with multiple keys are not yet supported.

## SYNOPSIS
<code><b>EXISTS key [key ...]</b></code><br>
This command is a predicate to check whether or not the given <code>key</code> exists.

## RETURN VALUE
Returns the number of existing keys.

## EXAMPLES
% <code>SET yuga1 "Africa"</code><br>
"OK"<br>
% <code>SET yuga2 "America"</code><br>
"OK"<br>
% <code>EXISTS yuga1</code><br>
1<br>
% <code>EXISTS yuga1 yuga2 not_a_key</code><br>
2<br>

## SEE ALSO
[`del`](../del/), [`get`](../get/), [`getrange`](../getrange/), [`hdel`](../hdel/), [`hexists`](../hexists/), [`hget`](../hget/), [`hset`](../hset/), [`mget`](../mget/), [`mset`](../mset/), [`sadd`](../sadd/), [`set`](../set/)