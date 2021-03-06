.. list-table::
   :widths: 40 10 10 10 10 10 10
   :header-rows: 0

   * -
     - English
     - `Русский <https://regex.sorokin.engineer/ru/latest/tregexpr.html>`__
     - `Deutsch <https://regex.sorokin.engineer/de/latest/tregexpr.html>`__
     - `Български <https://regex.sorokin.engineer/bg/latest/tregexpr.html>`__
     - `Français <https://regex.sorokin.engineer/fr/latest/tregexpr.html>`__
     - `Español <https://regex.sorokin.engineer/es/latest/tregexpr.html>`__

TRegExpr
========

Implements `regular expressions <regular_expressions.html>`_ in pure pascal.
Is compatible with Free Pascal, Delphi 2-7, Borland C++ Builder 3-6.

To use it just copy `source code <https://github.com/andgineer/TRegExpr/blob/master/src/RegExpr.pas>`_
into your project.

The library had already included into
`Lazarus (Free Pascal) <http://wiki.freepascal.org/Regexpr>`_ project so you
do not need to copy anything if you use `Lazarus <https://www.lazarus-ide.org/>`_.

TRegExpr class
--------------

VersionMajor, VersionMinor
~~~~~~~~~~~~~~~~~~~~~~~~~~

Return major and minor version, for example, for ``version 0.944``

::

    VersionMajor = 0
    VersionMinor = 944

Expression
~~~~~~~~~~

Regular expression.

For optimization regular expression is automatically compiled into ``P-code``.
Human-readable form of the ``P-code`` returns by Dump_.

In case of any errors in compilation, ``Error`` method is called (by
default ``Error`` raises exception ERegExpr_)

ModifierStr
~~~~~~~~~~~

Set or get values of
`regular expression modifiers <regular_expressions.html#modifiers>`__.

Format of the string is similar as in
`(?ismx-ismx) <regular_expressions.html#inlinemodifiers>`__. For example
``ModifierStr := ‘i-x’`` will switch on modifier `/i <regular_expressions.html#i>`_,
switch off `/x <regular_expressions.html#x>`_ and leave unchanged others.

If you try to set unsupported modifier, ``Error`` will be called.

ModifierI
~~~~~~~~~

`Modifier /i, "case-insensitive" <regular_expressions.html#i>`, initialized with
RegExprModifierI_ value.

ModifierR
~~~~~~~~~

`Modifier /r, "Russian range extension" <regular_expressions.html#r>`_, initialized with
RegExprModifierR_ value.

ModifierS
~~~~~~~~~

`Modifier /s, "single line strings" <regular_expressions.html#s>`_,
initialized with RegExprModifierS_ value.

ModifierG
~~~~~~~~~

`Modifier /g, "greediness" <regular_expressions.html#g>`_, initialized
with RegExprModifierG_ value.

ModifierM
~~~~~~~~~

`Modifier /m, "multi-line strings" <regular_expressions.html#m>`_, initialized
with RegExprModifierM_ value.

ModifierX
~~~~~~~~~

`Modifier /x, "eXtended syntax" <regular_expressions.html#x>`_,
initialized with RegExprModifierX_ value.

Exec
~~~~

Match the regular expression against ``AInputString``.

Available overloaded ``Exec`` version without ``AInputString`` - it uses ``AInputString``
from previous call.

See also global function ExecRegExpr_ that you can use without explicit ``TRegExpr``
object creation.

ExecNext
~~~~~~~~

Find next match.

Without parameter works the same as

::

    if MatchLen [0] = 0
      then ExecPos (MatchPos [0] + 1)
      else ExecPos (MatchPos [0] + MatchLen [0]);

Raises exception if used without preceeding successful call to
Exec_, ExecPos_ or ExecNext_.

So you always must use something like

::

    if Exec (InputString)
      then
        repeat
          { proceed results}
        until not ExecNext;

ExecPos
~~~~~~~

Finds match for ``InputString`` starting from ``AOffset`` position

::

    AOffset = 1 // first char of InputString

InputString
~~~~~~~~~~~

Returns current input string (from last Exec_ call or last assign to this
property).

Any assignment to this property clears Match_, MatchPos_ and MatchLen_.

Substitute
~~~~~~~~~~

::

    function Substitute (const ATemplate : RegExprString) : RegExprString;

Returns ``ATemplate`` with ``$&`` or ``$0`` replaced by whole regular expression
and ``$n`` replaced by occurence of subexpression number ``n``.

To place into template characters ``$`` or ``\``, use prefix ``\``, like ``\\`` or ``\$``.

====== ===============================
symbol description
====== ===============================
``$&`` whole regular expression match
``$0`` whole regular expression match
``$n`` regular subexpression ``n`` match
``\n`` in Windows replaced with ``\r\n``
``\l`` lowcase one next char
``\L`` lowercase all chars after that
``\u`` uppcase one next char
``\U`` uppercase all chars after that
====== ===============================

::

     '1\$ is $2\\rub\\' -> '1$ is <Match[2]>\rub\'
     '\U$1\\r' transforms into '<Match[1] in uppercase>\r'

If you want to place raw digit after ‘$n’ you must delimit ``n`` with curly
braces ``{}``.

::

     'a$12bc' -> 'a<Match[12]>bc'
     'a${1}2bc' -> 'a<Match[1]>2bc'.

Split
~~~~~

Split AInputStr into APieces by r.e. occurencies

Internally calls Exec_ / ExecNext_

See also global function SplitRegExpr_ that you can use without explicit ``TRegExpr``
object creation.

.. _Replace:

Replace, ReplaceEx
~~~~~~~~~~~~~~~~~~

::

    function Replace (Const AInputStr : RegExprString;
      const AReplaceStr : RegExprString;
      AUseSubstitution : boolean= False)
     : RegExprString; overload;

    function Replace (Const AInputStr : RegExprString;
      AReplaceFunc : TRegExprReplaceFunction)
     : RegExprString; overload;

    function ReplaceEx (Const AInputStr : RegExprString;
      AReplaceFunc : TRegExprReplaceFunction):
      RegExprString;

Returns the string with r.e. occurencies replaced by the replace string.

If last argument (``AUseSubstitution``) is true, then ``AReplaceStr`` will
be used as template for Substitution methods.

::

    Expression := '((?i)block|var)\s*(\s*\([^ ]*\)\s*)\s*';
    Replace ('BLOCK( test1)', 'def "$1" value "$2"', True);

Returns ``def "BLOCK" value "test1"``

::

    Replace ('BLOCK( test1)', 'def "$1" value "$2"', False)

Returns ``def "$1" value "$2"``

Internally calls Exec_ / ExecNext_

Overloaded version and ``ReplaceEx`` operate with call-back function,
so you can implement really complex functionality.

See also global function ReplaceRegExpr_ that you can use without explicit ``TRegExpr``
object creation.

SubExprMatchCount
~~~~~~~~~~~~~~~~~

Number of subexpressions has been found in last Exec_ / ExecNext_ call.

If there are no subexpr. but whole expr was found (Exec\* returned
True), then ``SubExprMatchCount=0``, if no subexpressions nor whole r.e.
found (Exec_ / ExecNext_ returned false) then ``SubExprMatchCount=-1``.

Note, that some subexpr. may be not found and for such subexpr.
``MathPos=MatchLen=-1`` and ``Match=’’``.

::

    Expression := '(1)?2(3)?';
    Exec ('123'): SubExprMatchCount=2, Match[0]='123', [1]='1', [2]='3'

    Exec ('12'): SubExprMatchCount=1, Match[0]='12', [1]='1'

    Exec ('23'): SubExprMatchCount=2, Match[0]='23', [1]='', [2]='3'

    Exec ('2'): SubExprMatchCount=0, Match[0]='2'

    Exec ('7') - return False: SubExprMatchCount=-1


MatchPos
~~~~~~~~

pos of entrance subexpr. ``#Idx`` into tested in last ``Exec*`` string.
First subexpr. have ``Idx=1``, last - ``MatchCount``, whole r.e. have
``Idx=0``.

Returns ``-1`` if in r.e. no such subexpr. or this subexpr. not found in
input string.

MatchLen
~~~~~~~~

len of entrance subexpr. ``#Idx`` r.e. into tested in last ``Exec*``
string. First subexpr. have ``Idx=1``, last - MatchCount, whole r.e.
have ``Idx=0``.

Returns -1 if in r.e. no such subexpr. or this subexpr. not found in
input string.

Match
~~~~~

Returns ``’’`` if in r.e. no such subexpression or this subexpression
was not found in the input string.

LastError
~~~~~~~~~

Returns ``ID`` of last error, ``0`` if no errors (unusable if ``Error`` method
raises exception) and clear internal status into ``0`` (no errors).

ErrorMsg
~~~~~~~~

Returns ``Error`` message for error with ``ID = AErrorID``.

CompilerErrorPos
~~~~~~~~~~~~~~~~

Returns pos in r.e. there compiler stopped.

Useful for error diagnostics

SpaceChars
~~~~~~~~~~

Contains chars, treated as ``\s`` (initially filled with RegExprSpaceChars_
global constant)

WordChars
~~~~~~~~~

Contains chars, treated as ``\w`` (initially filled with RegExprWordChars_
global constant)


LineSeparators
~~~~~~~~~~~~~~

line separators (like ``\n`` in Unix), initially filled with
RegExprLineSeparators_ global constant)

see also `Line Boundaries <regular_expressions.html#lineseparators>`__

LinePairedSeparator
~~~~~~~~~~~~~~~~~~~

paired line separator (like ``\r\n`` in DOS and Windows).

must contain exactly two chars or no chars at all, initially filled with
RegExprLinePairedSeparator global constant)

see also `Line Boundaries <regular_expressions.html#lineseparators>`__

For example, if you need Unix-style behaviour, assign
``LineSeparators := #$a`` and ``LinePairedSeparator := ''`` (empty string).

If you want to accept as line separators only ``\x0D\x0A`` but not ``\x0D``
or ``\x0A`` alone, then assign ``LineSeparators := ''`` (empty string) and
``LinePairedSeparator := #$d#$a``.

By default ‘mixed’ mode is used (defined in
RegExprLine[Paired]Separator[s] global constants):

::

    LineSeparators := #$d#$a; 
    LinePairedSeparator := #$d#$a

Behaviour of this mode is detailed described in the
`Line Boundaries <regular_expressions.html#lineseparators>`__.

InvertCase
~~~~~~~~~~

Invertion of character case. Redefine it if you want different
behaviour.

Compile
~~~~~~~

Compiles regular expression.

Useful for example for GUI regular expressions editors - to check
regular expression without using it.

Dump
~~~~

Show ``P-code`` (compiled regular expression) as human-readable string.

Global constants
----------------

EscChar
~~~~~~~

Escape-char, by default ``\``.

RegExprModifierI
~~~~~~~~~~~~~~~~

`Modifier i <regular_expressions.html#i>`_ default value

RegExprModifierR
~~~~~~~~~~~~~~~~

`Modifier r <regular_expressions.html#r>`_ default value

RegExprModifierS
~~~~~~~~~~~~~~~~

`Modifier s <regular_expressions.html#s>`_ default value

RegExprModifierG
~~~~~~~~~~~~~~~~

`Modifier g <regular_expressions.html#g>`_ default value

RegExprModifierM
~~~~~~~~~~~~~~~~

`Modifier m <regular_expressions.html#m>`_ default value

RegExprModifierX
~~~~~~~~~~~~~~~~

`Modifier x <regular_expressions.html#x>`_ default value

RegExprSpaceChars
~~~~~~~~~~~~~~~~~

Default for SpaceChars_ property
 

RegExprWordChars
~~~~~~~~~~~~~~~~

Default value for WordChars_ property

 
RegExprLineSeparators
~~~~~~~~~~~~~~~~~~~~~

Default value for LineSeparators_ property

RegExprLinePairedSeparator
~~~~~~~~~~~~~~~~~~~~~~~~~~

Default value for LinePairedSeparator_ property


RegExprInvertCaseFunction
~~~~~~~~~~~~~~~~~~~~~~~~~

Default for InvertCase_ property

Global functions
----------------

All this functionality is available as methods of ``TRegExpr``, but with global functions
you do not need to create ``TReExpr`` instance so your code would be more simple if
you just need one function.

ExecRegExpr
~~~~~~~~~~~

true if the string matches the regular expression.
Just as Exec_ in ``TRegExpr``.

SplitRegExpr
~~~~~~~~~~~~

Splits the string by regular expressions.
See also Split_ if you prefer to create ``TRegExpr`` instance explicitly.

ReplaceRegExpr
~~~~~~~~~~~~~~

::

    function ReplaceRegExpr (
        const ARegExpr, AInputStr, AReplaceStr : RegExprString;
        AUseSubstitution : boolean= False
    ) : RegExprString; overload;

    Type
      TRegexReplaceOption = (rroModifierI,
                             rroModifierR,
                             rroModifierS,
                             rroModifierG,
                             rroModifierM,
                             rroModifierX,
                             rroUseSubstitution,
                             rroUseOsLineEnd);
      TRegexReplaceOptions = Set of TRegexReplaceOption;

    function ReplaceRegExpr (
        const ARegExpr, AInputStr, AReplaceStr : RegExprString;
        Options :TRegexReplaceOptions
    ) : RegExprString; overload;

Returns the string with regular expressions replaced by the ``AReplaceStr``.
See also Replace_ if you prefer to create TRegExpr instance explicitly.

If last argument (``AUseSubstitution``) is true, then ``AReplaceStr`` will
be used as template for ``Substitution methods``:

::

    ReplaceRegExpr (
      '((?i)block|var)\s*(\s*\([^ ]*\)\s*)\s*',
      'BLOCK(test1)',
      'def "$1" value "$2"',
      True
    )

Returns  ``def 'BLOCK' value 'test1'``

But this one (note there is no last argument):

::

    ReplaceRegExpr (
      '((?i)block|var)\s*(\s*\([^ ]*\)\s*)\s*',
      'BLOCK(test1)',
      'def "$1" value "$2"'
    )

Returns ``def "$1" value "$2"``

Version with options
^^^^^^^^^^^^^^^^^^^^

With ``Options`` you control ``\n`` behaviour (if ``rroUseOsLineEnd`` then ``\n`` is
replaced with ``\n\r`` in Windows and ``\n`` in Linux). And so on.

.. code-block:: pascal

    Type
      TRegexReplaceOption = (rroModifierI,
                             rroModifierR,
                             rroModifierS,
                             rroModifierG,
                             rroModifierM,
                             rroModifierX,
                             rroUseSubstitution,
                             rroUseOsLineEnd);

QuoteRegExprMetaChars
~~~~~~~~~~~~~~~~~~~~~

Replace all metachars with its safe representation, for example
``abc'cd.(`` converts into ``abc\'cd\.\(``

This function usefull for r.e. autogeneration from user input

RegExprSubExpressions
~~~~~~~~~~~~~~~~~~~~~

Makes list of subexpressions found in ``ARegExpr``

In ``ASubExps`` every item represent subexpression, from first to last, in
format:

 String - subexpression text (without ‘()’)

 low word of Object - starting position in ARegExpr, including ‘(’ if
exists! (first position is 1)

 high word of Object - length, including starting ‘(’ and ending ‘)’ if
exist!

``AExtendedSyntax`` - must be ``True`` if modifier ``/x`` will be ``On`` while
using the r.e.

Usefull for GUI editors of r.e. etc (you can find example of using in
`REStudioMain.pas <https://github.com/andgineer/TRegExpr/blob/74ab342b639fc51941a4eea9c7aa53dcdf783592/restudio/REStudioMain.pas#L474>`_)

=========== =======
Result code Meaning
=========== =======
0           Success. No unbalanced brackets was found
-1          there are not enough closing brackets ``)``
-(n+1)      at position n was found opening ``[`` without corresponding closing ``]``
n           at position n was found closing bracket ``)`` without corresponding opening ``(``
=========== ======= 

If ``Result <> 0``, then ``ASubExprs`` can contain empty items or illegal ones

ERegExpr
--------

::

    ERegExpr = class (Exception)
      public
       ErrorCode : integer; // error code. Compilation error codes are before 1000
       CompilerErrorPos : integer; // Position in r.e. where compilation error occured
     end;

Unicode
-------

UniCode slows down performance so use it only if you really need Unicode
support.

To use Unicode uncomment ``{$DEFINE UniCode}``
in `regexpr.pas <https://github.com/andgineer/TRegExpr/blob/29ec3367f8309ba2ecde7d68d5f14a514de94511/src/RegExpr.pas#L86>`__
(remove ``off``).


After that all strings will be treated as WideString.

 
