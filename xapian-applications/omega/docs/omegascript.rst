===========
OmegaScript
===========

OmegaScript adds processed text-generation commands to text templates
(which will usually be HTML, but can be XML or another textual format).
Commands take the form ``$command{comma,separated,arguments}`` or
``$simplecommand``, for example::

    <html>
    <head><title>Sample</title></head>
    <body>

    <p>
    You searched for '$html{$query}'.
    </p>

    </body>
    </html>

Where appropriate, arguments themselves can contain OmegaScript commands.
Where an argument is treated as a string, the string is precisely the contents
of that argument - there is no string delimiter (such as the double-quote
character '"' in C and similar languages).  This can make complex OmegaScript
slightly difficult to read at times.

When a command takes no arguments, the braces must be omitted (i.e.
`$msize` rather than `$msize{}` - the latter is a command with a single empty
argument).  If you want to have the value of `$msize` immediately
followed by a letter, digit, or "_", you can use an empty comment (`${}`) to
prevent the parser treating the following character as part of a command name.
E.g. `_$msize${}_` rather than `_$msize_`

It is important to realise that all whitespace is significant in OmegaScript
- e.g. if you put whitespace around a "," which separates two command arguments
then the whitespace will be part of the respective arguments.

Note that (by design) OmegaScript has no unbounded looping constructs.  You
can loop over entries in a list, but you can't loop until some arbitrary
condition is met.  This means that it's not possible to accidentally (or
deliberately!) write an OmegaScript template which contains an infinite loop.

OmegaScript literals
====================

::

    $$ - literal '$'
    $( - literal '{'
    $) - literal '}'
    $. - literal ','


OmegaScript commands
====================

In the following descriptions, a LIST is a string of tab-separated
values.

${...}
	commented-out code

$addfilter{TERM[,TYPE]}
        add filter term ``TERM`` as if it had been passed as a ``B`` CGI
        parameter (if ``TYPE`` is not-specified, empty or ``B``), or as a
        negated filter as if passed as an ``N`` CGI parameter (if ``TYPE`` is
        ``N``).  Invalid types result in an error.

        Support for the second parameter was added in Omega 1.4.12 - in older
        versions only ``$addfilter{TERM}`` was supported and added ``B``-style
        filters.

        You must use ``$addfilter`` before any command which requires the query
        to have been parsed - see ``$setmap`` for a list of these commands.

$allterms[{DOCID}]
        list of all terms indexing the document with docid `DOCID` - if used
        without a parameter list, them the docid of the current hit is
        implicitly used.

$cgi{CGI}
        lookup the value of a CGI parameter.  If the same parameter has
        multiple values, ``$cgi`` will pick one arbitrarily - use ``$cgilist``
        if you want all the values.

$cgilist{CGI}
	return a list of all values of a CGI parameter

$cgiparams
        return a list of all the unique CGI parameter names, sorted in
        ascending order by raw byte values.

$chr{CODEPOINT}
        return UTF-8 for the given Unicode codepoint, e.g. ``$chr{127866}``
        should display as a beer mug if the font has a suitable glyph.

        Since ASCII is a subset of Unicode, you can also produce control
        characters, e.g. ``$chr{13}`` gives a carriage return character.

        To convert a UTF-8 character to a Unicode codepoint, see ``$ord``.

        Added in Omega 1.3.4.

$collapsed
        number of other documents collapsed into current hit inside
        ``$hitlist``, which might be used like so::

             $if{$ne{$collapsed,0},at least $collapsed hidden results ($value{$cgi{COLLAPSE}})}

$contains{STRING1,STRING2}
        return position of first occurrence of STRING1 in STRING2, if present. Else return an empty string.
        Examples:

        ``$contains{fish,goldfish}`` gives ``"4"``

        ``$contains{fish,shark}`` gives ``""``

$csv{STRING[,ALWAYS_ESCAPE]}
        encode STRING for use as a field in a CSV file.  By default, escaping
        is done as described in RFC4180, except that we treat any byte value
        not otherwise mentioned as being 'TEXTDATA' (so %x00-%x09, %x0B-%x0C,
        %x0E-%x1F, %x7F-%xFF are also permitted there).  Examples:

        ``$csv{Safe in CSV!}`` gives ``Safe in CSV!``

        ``$csv{Not "safe"}`` gives ``"Not ""safe"""``

        ``$csv{3$. 2$. 1}`` gives ``"3, 2, 1"``

        Some CSV consumers don't follow the RFC, in which case you may need
        to encode additional values.  For this reason, ``$csv`` provides an
        highly conservative alternative mode in which any double quote
        characters in the string are doubled, and the result always wrapped in
        double quotes.  To select this mode, pass a second non-empty argument.
        Examples:

        ``$csv{Quote anyway,1}`` gives ``"Quote anyway"``

        ``$csv{Not "safe",1}`` gives ``"Not ""safe"""``

        Added in Omega 1.3.4.

$date{TIME_T[,FMT]}
	convert a time_t to strftime ``FMT`` (default: ``YYYY-MM-DD``).  The
	conversion is done in timezone UTC.

$dbname
	database name (multiple names are returned separated by "/").

$dbsize
	number of documents in the database (if multiple databases are being
	searched, this gives the total number).

$def{MACRONAME,VALUE}
	define a macro which can take 0 to 9 arguments.  You can call it with
        ``$MACRONAME`` (if it take 0 arguments) or
        ``$MACRONAME{ARG1,ARG2,ARG3}`` is it takes arguments.  In value,
        arguments are available as ``$1``, ``$2``, ...  ``$9``.  In the current
        implementation, macros can override OmegaScript commands, but this
        shouldn't be relied on.  It's recommended to use capitalised names for
        macros to avoid collision with future OmegaScript commands.

$defaultop
	"and" or "or" (set from CGI variable DEFAULTOP).

$emptydocs[{TERM}]
	returns a list of docids of any documents with document length zero
	(such documents probably only contain scanned images, rather than
	machine readable text, or suggest the input filter isn't working well).
	If TERM is specified, only consider documents matching TERM, otherwise
	all documents are considered (so Tapplication/pdf reports all PDF files
	for which no text was found).

	If you're using omindex, note that it skips files with zero size, so
	these won't get reported here as they aren't present in the database.

$env{VAR}
	lookup variable ``VAR`` in the environment.

$error
	error message (e.g. if a database wouldn't open, or the query couldn't
        be parsed, or a Xapian exception has been thrown) or empty if there
	wasn't an error.  You can set the error message yourself by using
        ``$seterror``.

$field{NAME[,DOCID]}
        lookup field ``NAME`` in document ``DOCID``.  If ``DOCID`` is omitted
        then the field is looked up in the current hit (which only works inside
        ``$hitlist``).

        If multiple instances of field exist the field values are returned as
        an OmegaScript list (i.e. tab separated), which means you can pass the
        results to other commands which take a list, such as ``$foreach``, e.g.
        ::

            $foreach{$field{keywords},<b>$html{$_}</b><br>}

$filesize{SIZE}
	pretty printed filesize (e.g. ``1 byte``, ``100 bytes``, ``2.1K``,
        ``4.0M``, ``1.3G``).  If ``SIZE`` is empty or a negative integer,
        expands to nothing.

$filters
        serialised version of filter-like settings (currently ``B``, ``N``,
        ``DATEVALUE``, ``START``, ``END``, ``SPAN``, ``COLLAPSE``,
        ``DOCIDORDER``, ``SORT``, ``SORTREVERSE``, ``SORTAFTER``, and
        ``DEFAULTOP``) - set ``xFILTERS`` to this so that Omega can detect when
        the filters have changed and force the first page.

$filterterms{PREFIX}
        list of all terms in the database with prefix ``PREFIX``, intended to
        be used to allow drop-down lists and sets of radio buttons to be
	dynamically generated, e.g.::

             Hostname:
             <SELECT NAME="B">
             <OPTION VALUE=""
             $if{$map{$cgilist{B},$eq{$substr{$_,0,1},H}},,SELECTED}> Any
             $foreach{$filterterms{H},
             <OPTION VALUE="$html{$_}" $if{$find{$cgilist{B},$_},SELECTED}>
             $html{$substr{$_,1}}
             </OPTION>
             }
             </SELECT>

$find{LIST,STRING}
        returns the number of the first entry in ``LIST`` which is equal to
        ``STRING`` (starting from 0) or the empty string if no entry matches.

$fmt
	name of current format (as set by CGI parameter ``FMT``, or the default)

$foreach{LIST,STUFF)
        evaluated argument ``STUFF`` for each entry in list ``LIST``. If
        ``LIST`` contains the entries ``15``, ``13``, ``5``, ``7``, ``1``
        then::

            "$foreach{LIST,$chr{$add{$_,64}}}" = "OMEGA"

        If you want a list as output instead then see ``$map``.

        Added in Omega 1.4.18.

$freq{term}
	frequency of a term

$hash{TEXT,HASH}
    unique ID for ``TEXT`` string using the hashing algorithm specified by
    ``HASH`` which must be a lowercase string. Currently, this command only
    supports MD5 which yields a 128-bit hash sequence serialised as 32
    hexadecimal characters.

$highlight{TEXT,LIST[,OPEN[,CLOSE]]}
	html escape string (<>&, etc) and highlight any terms from ``LIST``
        that appear in ``TEXT`` by enclosing them in ``OPEN`` and ``CLOSE``.
        If ``OPEN`` is specified, but close is omitted, ``CLOSE`` defaults to
        the appropriate closing tag for ``OPEN`` (i.e. with a "/" in front and
        any parameters removed).  If both are omitted, then ``OPEN`` is set to:
	``<b style="color:XXXXX;background-color:#YYYYYY">`` (where ``YYYYYY``
        cycles through ``ffff66`` ``99ff99`` ``99ffff`` ``ff66ff`` ``ff9999``
        ``990000`` ``009900`` ``996600`` ``006699`` ``990099`` and ``XXXXX``
        is ``black`` if ``YYYYYY`` contains an ``f``, and otherwise ``white``)
        and ``CLOSE`` is set to ``</b>``.

$hit
	MSet index of current doc (first document in MSet is 0, so if
	you want to number the hits 1, 2, 3, ... use ``$add{$hit,1}``).

$hitlist{STUFF}
        evaluate ``STUFF`` once for each match in the result list.  During
        each evaluation ``$field``, ``$id``, ``$percentage``, ``$score``,
        ``$terms``, ``$weight``, etc will report values for the current hit.

$hitsperpage
	hits per page (as set by ``HITSPERPAGE``, or the default)

$hostname{URL}
	return the hostname from url ``URL``

$html{TEXT}
	html escape string (``<>&"`` are escaped to ``&lt;``, etc).

$htmlstrip{TEXT}
	html strip tags from string (``<...>``, etc).

$httpheader{NAME,VALUE}
	specify an additional HTTP header to be generated by Omega.
	For example::

	 $httpheader{Cache-Control,max-age=0$.private}

	If ``Content-Type`` is not specified by the template, it defaults
	to ``text/html``.  Headers must be specified before any other
	output from the OmegaScript template - any ``$httpheader{}``
	commands found later in the template will be silently ignored.

$id
	document id of current document

$json{STRING}
        encode STRING as a JSON string (not including the enclosing quotes), e.g.
        ``$json{The path is "C:\"}`` gives ``The path is \"C:\\\"``

        Added in Omega 1.3.1.

$jsonarray{LIST[,FORMAT]}
        encodes LIST (a string of tab-separated values) as a JSON array.  By
        default the elements of the array are encoded as JSON strings, but
        if ``FORMAT`` is specified it's evaluated for each element in turn
        with ``$_`` set to the element value and the result used instead.

        The default ``FORMAT`` is equivalent to ``"$json{$_}"``.

        Examples:

        ``$jsonarray{$split{a "b" c:\}}`` gives ``["a","\"b\"","c:\\"]``

        ``$jsonarray{$split{2 3 5 7},$mul{$_,$_}}`` gives ``[4,9,25,49]``

        Added in Omega 1.3.1, but buggy until 1.3.4.

        Support for the second argument added in Omega 1.4.15.

$jsonbool{COND}
        returns a JSON bool value (i.e. ``true`` or ``false``) for OmegaScript
        value ``COND``.

        This is exactly equivalent to ``$if{COND,true,false}`` and is provided
        just to allow more readable JSON-producing templates.  This means that
        ``COND`` being empty is false and all non-empty values are true (so
        note that ``$jsonbool{0}`` gives ``true`` - if you want a numeric test,
        you can use ``$jsonbool{$ne{VALUE,0}}``

        Added in Omega 1.4.15.

$jsonobject{MAP[,KEYFORMAT[,VALUEFORMAT]]}
        encodes OmegaScript map ``MAP`` (as set by ``$setmap``) as a JSON object.

        ``KEYFORMAT`` provides a way to modify key values.  It's evaluated for
        each key with ``$_`` set to the OmegaScript map key.  If omitted or
        empty then the keys are used as-is (so it effectively defaults to
        ``$_``).  For example ``$jsonobject{foo,$lower{$_}}`` forces keys to
        lower case.

        You probably want to avoid creating duplicate keys (RFC 2119 says they
        ``SHOULD be unique``).  Note that the resulting value should be an
        OmegaScript string - don't pass it though ``$json{}`` or wrap it in
        double quotes.

        ``VALUEFORMAT`` provides a way to specify how to encode values.  It's
        evaluated for each value with ``$_`` set to the OmegaScript map value
        and the result should be JSON to use as the JSON object value.  If
        omitted or empty the value is encoded as a JSON string (so effectively
        the default is ``"$json{$_}"``).  Note that (unlike ``KEYFORMAT``) this
        does need to include ``$json{}`` and double quotes, because the value
        doesn't have to be a JSON string.

        Simple example::

          $jsonobject{foo}

        More complex example which upper-cases the keys and uses JSON integers
        for the values::

          $jsonobject{foo,$upper{$_},$_}

        Added in Omega 1.4.15.  Since 1.4.19 the returned JSON no longer
        contains newlines, which makes it usable as a single line serialisation
        format without post-processing.

$keys{MAP}
        returns a list containing the keys of MAP (as set by ``$setmap``).
        The keys are in sorted order (by raw byte comparison).

        Added in Omega 1.4.15.

$last
        MSet index one beyond the end of the current page (so ``$hit`` runs
        from ``0`` to ``$sub{$last,1}``).

$lastpage
	number of last page of hits (may be an underestimate unless
	``$thispage`` == ``$lastpage``).

$length{LIST}
	number of entries in ``LIST``.

$list{LIST,...}
	pretty print list. If ``LIST`` contains 1, 2, 3, 4 then::

	 "$list{LIST,$. }" = "1, 2, 3, 4"
	 "$list{LIST,$. , and }" = "1, 2, 3 and 4"
	 "$list{LIST,List ,$. ,.}" = "List 1, 2, 3, 4."
	 "$list{LIST,List ,$. , and ,.}" = "List 1, 2, 3 and 4."

	NB ``$list`` returns an empty string for an empty list (so the
	last two forms aren't redundant as it may at first appear).

$log{LOGFILE[,ENTRY]}
        append to the log file ``LOGFILE``.  ``LOGFILE`` will be resolved as a
        relative path starting from directory ``log_dir`` (as specified in
        ``omega.conf``).  ``LOGFILE`` may not contain the substring ``..``.

        ``ENTRY`` is the OmegaScript for the log entry, which is evaluated and
        a linefeed appended.  ``ENTRY`` defaults to a format similar to the
        Common Log Format used by webservers.  If an error occurs when trying
        to open the log file then ``ENTRY`` won't be evaluated.

        If the logfile can't be opened or writing to it fails then ``$log``
        returns an error message (since Omega 1.5.0), otherwise it returns
        nothing.  If you want to ignore logging errors, you can ignore the
        return value using ``$if`` with no action like so::

         $if{$log{example.log}}

$lookup{CDBFILE,KEY}
        Return the tag corresponding to key ``KEY`` in the CDB file
        ``CDBFILE``.  If the file doesn't exist, or ``KEY`` isn't a key in it,
        then ``$lookup`` expands to nothing.  CDB files are compact disk based
        hashtables.  For more information and public domain software which can
        create CDB files, please visit: http://www.corpit.ru/mjt/tinycdb.html

	An example of how this might be used is to map top-level domains to
	country names.  Create a CDB file tld_en which maps "fr" to "France",
	"de" to "Germany", etc and then you can translate a country code to
	the English country name like so::

	 "$or{$lookup{tld_en,$field{tld}},.$field{tld}}"

	If a tld isn't in the CDB (e.g. "com"), this will expand to ".com".

	You can take this further and prepare a set of CDBs mapping tld codes
	to names in other languages - tld_fr for French, tld_de for German.
        Then if you have the ISO language code in ``$opt{lang}`` you can
        replace ``tld_en`` with ``tld_$or{$opt{lang},en}`` and automatically
        translate into the currently set language, or English if no language is
        set.

$lower{TEXT}
	return UTF-8 text ``TEXT`` converted to lower case.

$map{LIST,STUFF)
        map a list into the evaluated argument. If ``LIST`` contains ``1``,
        ``2`` then::

            "$map{LIST,x$_=$_;}" = "x1=1;	x2=2;"

        Note that $map{} returns a list (since Omega 0.5.0). If the tabs are a
        problem, then ``$foreach{LIST,STUFF}`` does the same thing but just
        concatenates the results directly rather than adding tabs to make a
        list.

$match{REGEX,STRING[,OPTIONS]}
	perform a regex match using Perl-compatible regular expressions. Returns
	true if a match is found, else it returns an empty string.

	The optional OPTIONS argument can contain zero or more of the letters
	``imsx``, which have the same meanings as the corresponding Perl regexp
	modifiers:

	* ``i`` - make the pattern matching case-insensitive
	* ``m`` - make ``^``/``$`` match after/before embedded newlines
	* ``s`` - allows ``.`` in the pattern to match a linefeed
	* ``x`` - allow whitespace and ``#``-comments in the pattern

$msize
	estimated number of matches.

$msizeexact
        return ``true`` if ``$msize`` is exact (or "" if it is estimated).
        Exactly equivalent to: ``$eq{$msizelower,$msizeupper}``

$msizelower
        lower bound on number of matches.

$msizeupper
        upper bound on number of matches.

$nice{number}
	pretty print integer (with thousands separator).

$now
	number of seconds since the epoch (suitable for feeding to ``$date``).
	Whether ``$now`` returns the same value for repeated calls in the same
	Omega search session is unspecified.

$opt{OPT}
	lookup an option value (as set by ``$set``).

$opt{MAP,OPT}
	lookup an option within a map (as set by ``$setmap``).

$ord{STRING}
        return codepoint for first character of UTF-8 string.  If the argument
        is an empty string, then an empty string is returned.

        For example, ``$ord{One more time}`` gives ``79``.

        To convert a Unicode code point into a UTF-8 string, see ``$chr``.

        Added in Omega 1.3.4.

$pack{NUMBER}
	converts a number to a 4 byte big-endian binary string

$percentage
	percentage score of current hit (in range 1-100).

	You probably don't want to show these percentage scores to end
	users in new applications - they're not really a percentage of
	anything meaningful, and research seems to suggest that users
	don't find numeric scores in search results useful.

$prettyterm{TERM}
	convert a term to "user form", as it might be entered in a query.  If
	a matching term was entered in the query, just use that (the first
	occurrence if a term was generated multiple times from a query).
	Otherwise term prefixes are converted back to user forms as specified
	by ``$setmap{prefix,...}`` and ``$setmap{boolprefix,...}``.

$prettyurl{URL}
	Prettify URL.  This command undoes RFC3986 URL escaping which doesn't
	affect semantics in practice, in order to make a prettier version of a
	URL for displaying to the user (rather than in links), but which should
	still work if copied and pasted.

$query[{PREFIX}]
	list of query strings for prefix PREFIX.  Any tab characters in the
	query strings are converted to spaces before adding them to the list
	(since an OmegaScript list is a string with tabs in).

	If PREFIX is omitted or empty, this is built from CGI ``P`` variable(s)
	plus possible added terms from ``ADD`` and ``X``.

	If PREFIX is non-empty, this is built from CGI ``P.PREFIX`` variables.

	Note: In Omega < 1.3.3, $query simply joins together the query strings
	with spaces rather than returning a list.

$querydescription
        a human readable description of the ``Xapian::Query`` object which
        omega builds.  Mostly useful for debugging omega itself.

$queryterms
	list of parsed query terms.

$range{START,END}
	return list of values between ``START`` and ``END``.

$random{HIGH}
	return a random value in the range [0, ``HIGH``].

$record[{ID}]
	raw record contents of document ``ID``.

$relevant[{ID}]
	document id ``ID`` if document is relevant, "" otherwise
	(side-effect: removes id from list of relevant documents
	returned by ``$relevants``).

$relevants
	return list of relevant documents

$score
	score (0-10) of current hit (equivalent to ``$div{$percentage,10}``).

$set{OPT,VALUE}
	set option value which may be looked up using ``$opt``.  You can use
	options as variables (for example, to store values you want to reuse
	without recomputing).  There are also several which Omega looks at
	and which you can set or use:

	* decimal - the decimal separator ("." by default - localised query
	  templates may want to set this to ",").
	* thousand - the thousands separator ("," by default - localised query
	  templates may want to set this to ".", " ", or "").
	* stemmer - which stemming language to use ("english" by default, other
	  values are as understood by ``Xapian::Stem``, so "none" means no
	  stemming).
        * stem_strategy - tell the query parser how to apply the stemmer - can
          be one of:

          + ``all``: stem all terms
          + ``all_z``: stem all terms and add a Z prefix
          + ``none``: don't stem any terms (ignoring any stemmer set)
          + ``some``: the default
          + ``some_full_pos``: like ``some`` but assume positional data has
            been stored for stemmed terms too.

          Unknown values are ignored.  Added in Omega 1.4.8.
	* stem_all - if "true", then tell the query parser to stem all words,
          even capitalised ones.  Now deprecated in favour of setting
          ``stem_strategy`` to ``all``, and ignored if ``stem_strategy`` is
          also set.
	* fieldnames - if set to a non-empty value then the document data is
	  parsed with each line being the value of a field, and the names
	  are taken from entries in the list in fieldnames.  So
          ``$set{fieldnames,$split{title sample url}}`` will take the first
          line as the "title" field, the second as the "sample" field and the
	  third as the "url" field.  Any lines without a corresponding field
	  name will be ignored.  If unset or empty then the document data is
	  parsed as one field per line in the format NAME=VALUE (where NAME is
	  assumed not to contain '=').
        * weighting - set the weighting scheme to use, and (optionally) the
          parameters to use if the weighting scheme supports them.  The syntax
          is a string consisting of the scheme name followed by any parameters,
          all separated by whitespace.  Any parameters not specified will use
          their default values.  Valid scheme names are
          ``bb2`` (in Omega >= 1.3.2), ``bm25``, ``bool``,
          ``coord`` (in Omega >= 1.4.1),
          ``dlh`` (in Omega >= 1.3.2), ``dph`` (in Omega >= 1.3.2),
          ``ifb2`` (in Omega >= 1.3.2), ``ineb2`` (in Omega >= 1.3.2),
          ``inl2`` (in Omega >= 1.3.2), ``lm`` (in Omega >= 1.3.2),
          ``pl2`` (in Omega >= 1.3.2), ``tfidf`` (in Omega >= 1.3.1),
          and ``trad``.  e.g.  ``$set{weighting,bm25 1 0.8}``

        * expansion - set the query expansion scheme to use, and (optionally)
          the parameters to use if the expansion scheme supports them. The syntax
          is a string consisting of the scheme name followed by any parameters,
          all separated by whitespace.  Any parameters not specified will use
          their default values.  Valid expansion schemes names are
          ``trad`` and ``bo1``.  e.g.
          ``$set{expansion,trad 2.0}``
        * weightingpurefilter - normally a query consisting only of filter
          terms won't have relevance weights calculated.  This option allows
          you to specify a weighting scheme to use for such queries, with the
          same values supported as for ``weighting`` above.  For example,
          ``$set{weightingpurefilter,coord}`` will weight such queries by
          how many filter terms match each document.

	Omega 1.2.5 and later support the following options, which can be set
	to a non-empty value to enable the corresponding ``QueryParser`` flag.
	Omega sets ``flag_default`` to ``true`` by default - you can set it to
	an empty value to turn it off (``$set{flag_default,}``):

	* flag_auto_multiword_synonyms
	* flag_auto_synonyms
	* flag_boolean
	* flag_boolean_any_case
	* flag_cjk_ngram (new in 1.2.22 and 1.3.4)
	* flag_cjk_words (new in 1.5.0)
	* flag_default
	* flag_fuzzy (new in 1.5.0)
	* flag_lovehate
	* flag_partial
	* flag_phrase
	* flag_pure_not
	* flag_spelling_correction (see ``$suggestion`` for suggested
	  correction)
	* flag_synonym
	* flag_wildcard
        * flag_wildcard_glob (new in 1.5.0)
        * flag_wildcard_multi (new in 1.5.0)
        * flag_wildcard_single (new in 1.5.0)

        Note that the ``Xapian::QueryParser::FLAG_ACCUMULATE`` flag is always
        enabled by Omega because it's needed for ``$stoplist`` and ``$unstem``
        to work correctly, and is deliberately not included in the above list.

	Omega 1.2.7 added support for parsing different query fields with
	different prefixes and you can specify different QueryParser flags for
	each prefix - for example, for the ``XFOO`` prefix use
	``XFOO:flag_pure_not``, etc.  The unprefixed constants provide a
	default value for these.  If a flag is set in the default, the prefix
	specific flag can unset it if it is set to the empty value (e.g.
	``$set{flag_pure_not,1}$set{XFOO:flag_pure_not,}``).

	You can use ``:flag_partial``, etc to set or unset a flag just for
	unprefixed fields.

	Similarly, ``XFOO:stemmer`` specifies the stemmer to use for field
	``XFOO``, with ``stemmer`` providing a default.

$seterror{ERROR_MESSAGE}
	set error message for the current execution, which can also be looked
	up using ``$error``.

	Using ``$seterror`` error early in template prevents running the query.

	For example, ``$seterror`` can be used when the user enters a wrong
	parameter in the search.

$setrelevant{docids}
	add documents into the RSet

$setmap{MAP,NAME1,VALUE1,...}
	set a map of option values which may be looked up against using
	``$opt{MAP,NAME}`` (maps with the same name are merged rather than
	the old map being completely replaced).

	You can create and use of maps in your own templates, but Omega also
	has several standard maps used to control building the query:

	Omega uses the "prefix" map to set the prefixes understood by the query
	parser.  So if you wish to translate a prefix of "author:" to A and
	"title:" to "S" you would use::

	 $setmap{prefix,author,A,title,S}

	In Omega 1.3.0 and later, you can map a prefix in the query string to
	more than one term prefix by specifying an OmegaScript list, for
	example to search unprefixed and S prefix by default use this
	(this also shows how you can map from an empty query string prefix, and
	also that you can map to an empty term prefix - these don't require
	Omega 1.3.0, but become much more useful in combination with this new
	feature)::

	 $setmap{prefix,,$split{ S}}

	Similarly, if you want to be able to restrict a search with a
	boolean filter from the text query (e.g. "group:" to "G") you
	would use::

	 $setmap{boolprefix,group,G}

	Don't be tempted to add whitespace around the commas, unless you want
	it to be included in the names and values!

	Another map (added in Omega 1.3.4) allows specifying any boolean
	prefixes which are non-exclusive, i.e. multiple filters of that
	type should be combined with ``OP_AND`` rather than ``OP_OR``.
	For example, if you have have a boolean filter on "material" using
	the ``XM`` prefix, and the items being searched are made of multiple
	materials, you likely want multiple material filters to restrict to
	items matching all the materials (the default it to restrict to any
	of the materials).  To specify this use
	``$setmap{nonexclusiveprefix,XM,true}`` (any non-empty value can
	be used in place of ``true``) - this feature affect both filters
	from ``B`` CGI parameters (e.g. ``B=XMglass&B=XMwood`` and those
	from parsing the query (e.g. ``material:glass material:wood`` if
	``$setmap{boolprefix,material,XM}`` is also in effect).

	Note: you must set the prefix-related maps before the query is parsed.
	This is done as late as possible - the following commands require the
	query to be parsed: $prettyterm, $query, $querydescription, $queryterms,
	$relevant, $relevants, $setrelevant, $unstem, and also these commands
	require the match to be run which requires the query to be parsed:
	$freqs, $hitlist, $last, $lastpage, $msize, $msizeexact, $terms,
	$thispage, $time, $topdoc, $topterms.

$slice{LIST,POSITIONS}
	returns the elements from ``LIST`` at the positions listed in the
	second list ``POSITIONS``.  The first item is at position 0.
	Any positions which are out of range will be ignored.

	For example, if ``LIST`` contains a, b, c, d then::

	 "$slice{LIST,2}" = "c"
	 "$slice{LIST,1	3}" = "b	d"
	 "$slice{LIST,$range{1,3}}" = "b	c	d"
	 "$slice{LIST,$range{-10,10}}" = "a	b	c	d"

$snippet{TEXT[,LENGTH[,FLAGS[,BRA[,KET[,GAP]]]]]}
        Generate a context-sensitive snippet from ``TEXT`` using the C++ API
        ``Xapian::MSet::snippet()`` suitable for inserting into an HTML or XML
        document.

        The snippet will be at most ``LENGTH`` bytes long (default: 200 if not
        specified or ``LENGTH`` is an empty string).  ``BRA`` and ``KET`` are
        added around matching terms in the sample (defaults: ``<strong>`` and
        ``</strong>``) and ``GAP`` is used to mark omitted parts of ``TEXT``
        (default: ``...``).

        ``FLAGS`` contains zero or more of the flags that are supported by the
        C++ API separated by ``|`` - here they are specified as
        case-insensitive strings, and the leading ``SNIPPET_`` is optional.

        For example::

        $snippet{$field{sample},,background_model|empty_without_match,<b>,</b>}

$sort{LIST[,OPTIONS]}
        sort the entries in a list.  The sort order is an ascending string sort
        by byte value by default.  ``OPTIONS`` is zero or more of the following
        characters which control the sort operation:

        * ``#`` : "natural number" sort suitable for use when generating
          drop-down lists.  Embedded digit sequences are handled specially:
          they are compared numerically, and sort before non-digits at the same
          point.  Digit sequences with the same numeric value are sorted
          such that the sequence with more leading zeros comes first (so when
          used with ``u`` only identical entries are removed).
        * ``r`` : reverse the sort order
        * ``u`` : output only the first (in input order) of an equal run
        * ``n`` : sort by string numerical value - the start of each entry is
          parsed as zero or more whitespace characters, an optional ``-``, zero
          or more digits, optionally followed by ``$opt{decimal}`` then zero or
          more digits.  Entries are regarded as equal if the numbers are equal
          and so only the first is kept with ``u``.  When ``u`` is not used,
          the order within groups of equal entries is resolved with a string
          sort.

        Options ``#`` and ``n`` aren't valid together.

$split{STRING}

$split{SPLIT,STRING}
	returns a list by splitting the string ``STRING`` into elements at each
        occurrence of the substring ``SPLIT``.  If ``SPLIT`` isn't specified,
        it defaults to a single space.  If ``SPLIT`` is empty, ``STRING`` is
        split into individual bytes.

	For example::

	 "$split{one two three}" = "one	two	three"

$srandom{SEED}
	``SEED`` specifies a seed for random number generation.

$stoplist
	returns a list of any terms in the query which were ignored as
	stopwords.  Since Omega 1.4.18 ``$stoplist`` reports such terms for
        all query strings parsed as part of the current query - in previous
        versions only this only reported stopwords for the query string
        which was parsed last (which would be the one with the prefix sorting
        last in byte sort order, and if there were multiple such query strings
        then the one specified last).

$subdb[{DOCID}]
        return the name from a ``DB`` parameter for the sub-database containing
        ``DOCID``.

        If ``DOCID`` is omitted it defaults to the current document in the
        hitlist.

        Prior to Xapian 1.4.12 the implementation assumed that each omega
        database name corresponded to a single Xapian database and if a
        database name referred to a stub database file expanding to multiple
        Xapian databases then this command would misbehave.  In 1.4.12 and
        later this case is taken into account.

$subid[{DOCID}]
        return the docid in ``$subdb{DOCID}`` corresponding to ``DOCID`` in the
        combined database.

        If ``DOCID`` is omitted it defaults to the current document in the
        hitlist.

        Prior to Xapian 1.4.12 the implementation assumed that each omega
        database name corresponded to a single Xapian database and if a
        database name referred to a stub database file expanding to multiple
        Xapian databases then this command would misbehave.  In 1.4.12 and
        later this case is taken into account.

$substr{STRING,START[,LENGTH]}
        returns the substring of ``STRING`` which starts at byte position
        ``START`` (the start of the string being 0) and is ``LENGTH`` bytes
        long (or to the end of ``STRING`` if ``STRING`` is less than
        ``START``+``LENGTH`` bytes long).  If ``LENGTH`` is omitted, the
        substring from ``START`` to the end of ``STRING`` is returned.

	If ``START`` is negative, it counts back from the end of ``STRING`` (so
	``$substr{hello,-1}`` is ``o``).

	If LENGTH is negative, it instead specifies the number of bytes
	to omit from the end of STRING (so "$substr{example,2,-2}" is "amp").
	Note that this means that "$substr{STRING,0,N}$substr{STRING,N}" is
	"STRING" whether N is positive, negative or zero.

$suggestion
        if ``$set{flag_spelling_correction,true}`` was done before the query
        was parsed, then ``$suggestion`` will return any suggested spelling
        corrected version of the query string.  If there are no spelling
        corrections, it will return an empty string.

$termprefix{TERM}
        return the prefix (if any) from a term.  Added in Omega 1.4.6.

$terms[{PREFIX}]
        list of query terms matching the current hit.  The ability to specify a
        prefix was added in Omega 1.3.5.  If no prefix is specified (i.e.
        ``$terms``), then only terms from the query string(s) are returned.
        This is different to an empty prefix (i.e. ``$terms{}``) which returns
        all query terms matching the current hit, so also includes filter
        terms.

$thispage
	page number of current page.

$time
	how long the match took (in seconds) e.g. ``0.078534``.  If no timing
	information was available, returns an empty value.

$topdoc
	first document on current page of hit list (counting from 0)

$topterms[{N}]
	list of up to ``N`` top relevance feedback terms (default 16)

$transform{REGEXP,SUBST,STRING[,OPTIONS]}
	transform string using Perl-compatible regular expressions.  This
	command is sort of like the Perl code::

         my $string = STRING;
         $string =~ s/REGEXP/SUBST/;
         print $string;

        In SUBST, ``\1`` to ``\9`` are substituted by the 1st to 9th bracket
        grouping (or are empty if there is no such bracket grouping).  ``\\``
        is a literal backslash.

        The optional OPTIONS argument is supported by Omega 1.3.4 and later.
        It can contain zero or more of the letters ``gimsx``, which have the
        same meanings as the corresponding Perl regexp modifiers:

         * ``g`` - replace all occurrences of the pattern in the string
         * ``i`` - make the pattern matching case-insensitive
         * ``m`` - make ``^``/``$`` match after/before embedded newlines
         * ``s`` - allows ``.`` in the pattern to match a linefeed
         * ``x`` - allow whitespace and ``#``-comments in the pattern

$truncate{STRING,LEN[,IND[,IND2]]}
	truncate STRING to LEN bytes, but try to break after a word (unless
	that would mean truncating to much less than LEN).  If we have to
	split a word, then IND is appended (if specified).  If we have to
	truncate (but don't split a word) then IND2 is appended (if specified).
	For example::

	 $truncate{$field{text},500,..., ...}

$uniq{LIST}
        remove adjacent duplicates, for example from an already sorted list
        (similar to the Unix ``uniq`` command line tool).

$unique{LIST}
        remove duplicates from a list - unlike ``$uniq``, duplicates don't
        need to be adjacent.  The first of each entry is kept, and order is
        preserved.  If the input list is already sorted then ``$uniq`` is
        more efficient.

$unpack{BINARYSTRING}
	converts a 4 byte big-endian binary string to a number, for example::

         $date{$unpack{$value{0}}}

$unprefix{TERM}
        remove the prefix (if any) from a term.  Added in Omega 1.4.6.

$unstem{TERM}
	maps a stemmed term to a list of the unstemmed forms of it used in
	the query.  Since Omega 1.4.18 ``$unstem`` reports unstemmed forms in
        all query strings parsed as part of the current query - in previous
        versions only this only reported unstemmed forms from the query string
        which was parsed last (which would be the one with the prefix sorting
        last in byte sort order, and if there were multiple such query strings
        then the one specified last).

$upper{TEXT}
	return UTF-8 text ``TEXT`` converted to upper case.

$url{TEXT}
	url encode argument

$value{VALUENO[,DOCID]}
        returns value number ``VALUENO`` for document ``DOCID``.  If ``DOCID``
        is omitted then the current hit is used (which only works inside
        ``$hitlist``).

$version
	omega version string - e.g. "xapian-omega 1.2.6"

$weight
	raw document weight of the current hit, as a floating point value
	(mostly useful for debugging purposes).

Numeric Operators:
==================

For Numeric Operators we allow trailing characters and also allow non-number arguments.
Reason for this behaviour is that it makes things robust if some of the parameters come in as CGI parameters.
Ex:- if you had a CGI param at the end of a URL that was supposed to be a number, and sent it in an email
or a message, it's possible for the person receiving to end up with a URL with a dot or semicolon at the
end (from punctuation in your message).

$add{...}
	add arguments together (if called with one argument, this will convert
	it to an integer and back, which ensures it is an integer).

$div{A,B}
	returns int(A / B) (or the text "divide by 0" if B is zero)

$mod{A,B}
	returns int(A % B) (or the text "divide by 0" if B is zero)

$max{A,...}
	maximum of the arguments

$min{A,...}
	minimum of the arguments

$mul{A,B,...}
	multiply arguments together

$muldiv{A,B,C}
	returns int((A * B) / C) (or the text "divide by 0" if C is zero)

$sub{A,B}
	returns (A - B)

Logical Operators:
==================

For Logical Operators we allow empty arguments.
Reason is that logical operators compare their arguments based on whether they are empty or not.
OmegaScript treats an empty string as a "false" logical value and any non-empty string as "true".

$and{...}
	logical short-cutting "and" of its arguments - evaluates
	arguments until it finds an empty one (and returns "") or
	has evaluated them all (returns "true")

$eq{A,B}
	returns "true" if A and B are the same, "" otherwise.

$ge{A,B}
	returns "true" if A is numerically >= B.

$gt{A,B}
	returns "true" if A is numerically > B.

$le{A,B}
	returns "true" if A is numerically <= B.

$lt{A,B}
	returns "true" if A is numerically < B.

$ne{A,B}
	returns "true" if A and B are not the same, "" if they are.

$not{A}
	returns "true" for the empty string, "" otherwise.

$or{...}
	logical short-cutting "or" of its arguments - returns first
	non-empty argument

Control:
========

$cond{COND1,THEN1[,COND2,THEN2]...[,ELSE]}
	evaluates ``COND1``, ``COND2``, ... in turn until a non-empty value is
        obtained, and then evaluates and returns the corresponding ``THEN``.
        If all ``COND`` values expand to empty values, then evaluates and
        returns ``ELSE`` (if present, otherwise returns nothing).

        ``$cond`` provides a neater way of writing a cascading series of
        ``$if`` checks.  If there's only one condition, ``$cond`` is equivalent
        to ``$if``.

        Added in Omega 1.4.6.

$if{COND[,THEN[,ELSE]]}
        if ``COND`` is non-empty, evaluates and returns ``THEN``; otherwise
        evaluates and returns ``ELSE``.  If ``THEN`` and/or ``ELSE`` are omitted
        then returns nothing.  You can use ``$if{COND}`` to evaluate ``COND``
        but discard the result of that evaluation, which can be useful if
        ``COND`` has side-effects.

        The ability to omit ``THEN`` was added in Omega 1.4.15.

$include{FILE[,FALLBACK]}
        include another OmegaScript file ``FILE``.  If opening ``FILE`` fails, then
        ``FALLBACK`` is evaluated and returned.

        Support for the ``FALLBACK`` argument was added in Omega 1.4.18.

$switch{EXPR,CASE1,VALUE1,[CASE2,VALUE2]...[,DEFAULT]}
        first evaluates ``EXPR``, and then evaluates ``CASE1``, ``CASE2``, ...
        in turn until one of them has the same value as ``EXPR`` did, and then
        evaluates and returns the corresponding ``VALUE``.  If none of the
        ``CASE`` values matches, then evaluates and returns ``DEFAULT`` (if
        present, otherwise returns nothing).

        Added in Omega 1.4.6.
