datrie
======

Super-fast, efficiently stored Trie for Python (2.x and 3.x).
Uses `libdatrie`_.

.. _libdatrie: http://linux.thai.net/~thep/datrie/datrie.html

Installation
============

::

    pip install datrie

Usage
=====

Create a new trie capable of storing items with lower-case ascii keys::

    >>> import string
    >>> import datrie
    >>> trie = datrie.Trie(string.ascii_lowercase)

``trie`` variable is a dict-like object that can have unicode keys of
certain ranges and Python objects as values.

In addition to implementing the mapping interface, tries facilitate
finding the items for a given prefix, and vice versa, finding the
items whose keys are prefixes of a given string. As a common special
case, finding the longest-prefix item is also supported.

.. warning::

    For efficiency you must define allowed character range(s) while
    creating trie. ``datrie`` doesn't check if keys are in allowed
    ranges at runtime, so be careful! Invalid keys are OK at lookup time
    but values won't be stored correctly for such keys.

Add some values to it (datrie keys must be unicode; the examples
are for Python 2.x)::

    >>> trie[u'foo'] = 5
    >>> trie[u'foobar'] = 10
    >>> trie[u'bar'] = 'bar value'
    >>> trie.setdefault(u'foobar', 15)
    10

Check if u'foo' is in trie::

    >>> u'foo' in trie
    True

Get a value::

    >>> trie[u'foo']
    5

Find all prefixes of a word::

    >>> trie.prefixes(u'foobarbaz')
    [u'foo', u'foobar']

    >>> trie.prefix_items(u'foobarbaz')
    [(u'foo', 5), (u'foobar', 10)]

    >>> trie.iter_prefixes(u'foobarbaz')
    <generator object ...>

    >>> trie.iter_prefix_items(u'foobarbaz')
    <generator object ...>

Find the longest prefix of a word::

    >>> trie.longest_prefix(u'foo')
    u'foo'

    >>> trie.longest_prefix(u'foobarbaz')
    u'foobar'

    >>> trie.longest_prefix(u'gaz')
    KeyError: u'gaz'

    >>> trie.longest_prefix(u'gaz', default=u'vasia')
    u'vasia'

    >>> trie.longest_prefix_item(u'foobarbaz')
    (u'foobar', 10)

Check if the trie has keys with a given prefix::

    >>> trie.has_keys_with_prefix(u'fo')
    True

    >>> trie.has_keys_with_prefix(u'FO')
    False

Get all items with a given prefix from a trie::

    >>> trie.keys(u'fo')
    [u'foo', u'foobar']

    >>> trie.items(u'ba')
    [(u'bar', 'bar value')]

    >>> trie.values(u'foob')
    [10]

Save & load a trie (values must be picklable)::

    >>> trie.save('my.trie')
    >>> trie2 = datrie.Trie.load('my.trie')


Trie and BaseTrie
=================

There are two Trie classes in datrie package: ``datrie.Trie`` and
``datrie.BaseTrie``. ``datrie.BaseTrie`` is slightly faster and uses less
memory but it can store only integer numbers 0 <= x <= 2147483647.
``datrie.Trie`` is a bit slower but can store any Python object as a value;
it is the trie created by ``datrie.new`` by default.

If you don't need values or integer values are OK then use ``datrie.BaseTrie``::

    import datrie
    import string
    trie = datrie.BaseTrie(string.ascii_lowercase)

Performance
===========

Performance is measured for ``datrie.Trie`` against Python's dict with
100k unique unicode words (English and Russian) as keys and '1' numbers
as values.

``datrie.Trie`` uses about 5M memory for 100k words; Python's dict
uses about 22M for this according to my unscientific tests.

This trie implementation is 2-6 times slower than python's dict
on __getitem__. Benchmark results (macbook air i5 1.7GHz,
"1.000M ops/sec" == "1 000 000 operations per second")::

    Python 2.6:
    dict __getitem__: 6.024M ops/sec
    trie __getitem__: 2.272M ops/sec

    Python 2.7:
    dict __getitem__: 6.693M ops/sec
    trie __getitem__: 2.357M ops/sec

    Python 3.2:
    dict __getitem__: 3.628M ops/sec
    trie __getitem__: 1.980M ops/sec

    Python 3.3b1:
    dict __getitem__: 6.721M ops/sec
    trie __getitem__: 2.584M ops/sec

Looking for prefixes of a given word is almost as fast as
__getitem__ (results are for Python 3.2, they are even faster under
Python 2.x on my machine)::

    trie.iter_prefix_items (hits):      0.431M ops/sec
    trie.prefix_items (hits):           0.685M ops/sec
    trie.prefix_items loop (hits):      0.601M ops/sec
    trie.iter_prefixes (hits):          0.814M ops/sec
    trie.iter_prefixes (misses):        1.565M ops/sec
    trie.iter_prefixes (mixed):         1.461M ops/sec
    trie.has_keys_with_prefix (hits):   1.945M ops/sec
    trie.has_keys_with_prefix (misses): 2.625M ops/sec
    trie.longest_prefix (hits):         1.750M ops/sec
    trie.longest_prefix (misses):       1.569M ops/sec
    trie.longest_prefix (mixed):        1.662M ops/sec
    trie.longest_prefix_item (hits):    1.075M ops/sec
    trie.longest_prefix_item (misses):  1.058M ops/sec
    trie.longest_prefix_item (mixed):   1.083M ops/sec

Looking for all words starting with a given prefix is mostly limited
by overall result count (this can be improved in future because a
lot of time is spent decoding strings from utf_32_le to Python's
unicode)::

    trie.items(prefix="xxx"), avg_len(res)==415:        0.690K ops/sec
    trie.keys(prefix="xxx"), avg_len(res)==415:         0.721K ops/sec
    trie.values(prefix="xxx"), avg_len(res)==415:       2.151K ops/sec
    trie.items(prefix="xxxxx"), avg_len(res)==17:       15.841K ops/sec
    trie.keys(prefix="xxxxx"), avg_len(res)==17:        16.829K ops/sec
    trie.values(prefix="xxxxx"), avg_len(res)==17:      43.930K ops/sec
    trie.items(prefix="xxxxxxxx"), avg_len(res)==3:     71.620K ops/sec
    trie.keys(prefix="xxxxxxxx"), avg_len(res)==3:      77.067K ops/sec
    trie.values(prefix="xxxxxxxx"), avg_len(res)==3:    157.464K ops/sec
    trie.items(prefix="xxxxx..xx"), avg_len(res)==1.4:  116.869K ops/sec
    trie.keys(prefix="xxxxx..xx"), avg_len(res)==1.4:   128.392K ops/sec
    trie.values(prefix="xxxxx..xx"), avg_len(res)==1.4: 194.388K ops/sec
    trie.items(prefix="xxx"), NON_EXISTING:             1753.472K ops/sec
    trie.keys(prefix="xxx"), NON_EXISTING:              1797.559K ops/sec
    trie.values(prefix="xxx"), NON_EXISTING:            1705.695K ops/sec

Build time is worse than dict's; updates are quite fast::

    dict __setitem__ (updates): 3.489M ops/sec
    trie __setitem__ (updates): 1.862M ops/sec
    dict __setitem__ (inserts): 3.628M ops/sec
    trie __setitem__ (inserts): 0.050M ops/sec
    dict setdefault (updates):  2.575M ops/sec
    trie setdefault (updates):  1.600M ops/sec
    dict setdefault (inserts):  2.596M ops/sec
    trie setdefault (inserts):  0.050M ops/sec


Please take this benchmark results with a grain of salt; this
is a very simple benchmark and may not cover your use case.

Current Limitations
===================

* keys must be unicode (no implicit conversion for byte strings
  under Python 2.x, sorry);
* there are no iterator versions of keys/values/items (this is a current
  limitation of libdatrie);
* it doesn't work under pypy+MacOS X (some obscure error which I don't
  understand);
* library is not tested with narrow Python builds.

Contributing
============

Development happens at github and bitbucket:

* https://github.com/kmike/datrie
* https://bitbucket.org/kmike/datrie

The main issue tracker is at github.

Feel free to submit ideas, bugs, pull requests (git or hg) or
regular patches.

Running tests and benchmarks
----------------------------

Make sure `tox`_ is installed and run

::

    $ tox

from the source checkout. Tests should pass under python 2.6, 2.7, 3.2 and 3.3.

.. note::

    At the moment of writing the latest pip release (1.1) does not
    support Python 3.3; in order to run tox tests under Python 3.3
    find the "virtualenv_support" directory in site-packages
    (of the env you run tox from) and place an sdist zip/tarball of the newer
    pip (from github) there.

::

    $ tox -c tox-bench.ini

runs benchmarks.

If you've changed anything in the source code then
make sure `cython`_ is installed and run

::

    $ update_c.sh

before each ``tox`` command.

Please note that benchmarks are not included in the release
tar.gz's because benchmark data is large and this
saves a lot of bandwidth; use source checkouts from
github or bitbucket for the benchmarks.

.. _cython: http://cython.org
.. _tox: http://tox.testrun.org

Authors & Contributors
----------------------

* Mikhail Korobov <kmike84@gmail.com>

This module is based on `libdatrie`_ C library and is inspired by
`fast_trie`_ Ruby bindings, `PyTrie`_ pure Python implementation
and `Tree::Trie`_ Perl implementation; some docs and API ideas are
borrowed from these projects.

.. _fast_trie: https://github.com/tyler/trie
.. _PyTrie: https://bitbucket.org/gsakkis/pytrie
.. _Tree::Trie: http://search.cpan.org/~avif/Tree-Trie-1.9/Trie.pm

License
=======

Licensed under LGPL v2.1.
