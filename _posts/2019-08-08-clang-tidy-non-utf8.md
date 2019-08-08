---
layout: post
title:  "clang-tidy на кодовой базе в windows 1251"
date:   2019-08-08 17:00:00 -0400
categories: code
---

Начал использовать clang-tidy для статического анализа. Главная проблема с ним - это отстутствие документации. В итоге пришлось забирать исходники, по ним разбираться, как оно работает, и прогибать под свои нужды.

`HeaderFilterRegex` оказался whitelist-списком, по которому можно матчить и пути.

Но подстава в том, что если в скрытом заголовке нашлась ошибка, то она, хоть и скрывается, но уведомления о ней всё равно всплывают наверх. Пришлось добавить блокирование всплывания уведомлений о потушенных ошибках:
~~~~~
+++ /clang-tools-extra/clang-tidy/ClangTidyDiagnosticConsumer.cpp
   if (LastErrorWasIgnored && DiagLevel == DiagnosticsEngine::Note)
     return;
 
+  if (LastErrorRelatesToUserCode == false &&
+      DiagLevel == DiagnosticsEngine::Note)
+    return;
+
~~~~~
{: .language-cpp}

Кроме того, пришлось запретить сообщать об ошибках в коде, который нам не принадлежит. Ибо как-то повлиять на это мы всё равно не можем:
~~~~~
+++ /clang-tools-extra/clang-tidy/ClangTidyDiagnosticConsumer.cpp
     ClangTidyError::Level Level = ClangTidyError::Warning;
-    if (DiagLevel == DiagnosticsEngine::Error ||
+    if (/*DiagLevel == DiagnosticsEngine::Error ||*/
~~~~~
{: .language-cpp}

Кодировка исходных файлов может быть только UTF-8. С другой кодировкой clang в принципе не умеет работать. Но так как нам не надо код компилировать, и что там захочет вывестись - это неважно, можно по коду расставить фрагменты, затыкающие его анализ и воспринимающие cp-1251 как ascii:
~~~~~
+++ /clang/lib/AST/Expr.cpp
-  assert((ByteLength % CharByteWidth == 0) &&
+  assert((ByteLength % CharByteWidth == 0) || true &&

+++ /clang/lib/Lex/Lexer.cpp
     // Non-ASCII characters tend to creep into source code unintentionally.
     // Instead of letting the parser complain about the unknown token,
     // just diagnose the invalid UTF-8, then drop the character.
-    Diag(CurPtr, diag::err_invalid_utf8);
+    // Diag(CurPtr, diag::err_invalid_utf8);
 
+++ /clang/lib/Lex/LiteralSupport.cpp
   // If we see bad encoding for unprefixed string literals, warn and
   // simply copy the byte values, for compatibility with gcc and older
   // versions of clang.
-  bool NoErrorOnBadEncoding = isAscii();
+  bool NoErrorOnBadEncoding = true;// isAscii();
 
-  if (Diags) {
+  if (false && Diags) {
~~~~~
{: .language-cpp}

Этих патчей оказалось достаточно, чтобы оно нормально заработало на кодовой базе в cp-1251 и полностью скрывало сообщения об ошибках в проигнорированных исходниках.
