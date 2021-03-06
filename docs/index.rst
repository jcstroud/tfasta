.. Created by phyles-quickstart.
   Add some items to the toctree.

tfasta 
======

Introduction
------------

The tfasta python package simplifies working
with fasta, providing functionality
for both reading and writing fasta files.
The "t" in "tfasta" represents
"templated", which means that fasta parsing is
performed according to pre-defined or user-defined
templates::

    >>> from tfasta import fasta_parser, T_NR
    >>> fast = fasta_parser("cytb.fas", template=T_NR)
    >>> f = fast.next()
    >>> print f['gi']
    5524211
    >>> print f['accession']
    AAD44166.1
    >>> print f['sequence'][:60]
    LCLYTHIGRNIYYGSYLYSETWNTGIMLLLITMATAFMGYVLPWGQMSFWGATVITNLFS

This example parses records that follow the conventions
of the NCBI non-redundant database (nr).

More examples are given below.

Installation
------------

Install tfasta with `pip`_ (recommended) or `easy_install`_::

  sudo pip install tfasta

Optionally, download the source files from
http://pypi.python.org/pypi/tfasta/
and run the following commands in the source directory::

  python setup.py build
  sudo python setup.py install

Home Page & Repository
----------------------

  - Home Page: http://pypi.python.org/pypi/tfasta/
  - Documentation: http://pythonhosted.org/radialx/
  - Repository: https://github.com/jcstroud/tfasta/


Usage
-----

Reading Fasta Files
~~~~~~~~~~~~~~~~~~~

Reading fasta files is performed with the *fasta_parser()* function.
The following text is the first 2 records of the
file "``short-extended.fas``"::

    >gi|32033604|ref|ZP_00133915.1|
    ATGQVIGTFTVRNDNGLHARPSAVLVQTLKPFAAKVTVENLDRGTAPANAKSTMKVVALG
    ASQAHRLRFVAEGEDAQQAIEALAKAFVEGLGESVSFVPAVEDTIEGAAQPQAVESAKNF
    ANPTASEPTVEGQVEGTFVIQNEHGLHARPSAVLVNEVKKYNATIVVQNLDRNTQLVSAK
    SLMKIVALGVVKGHRLHFVATGDDAQKAIDGIGEAIAAGLGE
    >gi|1573424|gb|AAC22107.1|
    VEGAVVGTFTIRNEHGLHARPSANLVNEVKKFTSKITMQNLTRESEVVSAKSLMKIVALG
    VTQGHRLRFVAEGEDAKQAIESLGKAIANGLGENVSAVPPSEPDTIEIMGDQIHTPAVTE
    DDNLPANAIEAVFVIKNEQGLHARPSAILVNEVKKYNASVAVQNLDRNSQLVSAKSLMKI
    VALGVVKGTRLRFVATGEEAQQAIDGIGAVIESGLGE

Like any other fasta file, ``short-extended.fas`` may be parsed
with a single command::

    fast = fasta_parser(file_name)

For example::

    >>> from tfasta import fasta_parser
    >>> fast = fasta_parser("short-extended.fas")
    >>> f = fast.next()
    >>> print f['name']
    gi|32033604|ref|ZP_00133915.1|
    >>> print f['sequence'][:60]
    ATGQVIGTFTVRNDNGLHARPSAVLVQTLKPFAAKVTVENLDRGTAPANAKSTMKVVALG
    >>> f = fast.next()
    >>> print f['name']
    gi|1573424|gb|AAC22107.1|

In this example, the *fasta_parser()* function returns
an iterator of dictionaries ("``fast``") with two
keys: ``name`` and ``sequence``.
The ``name`` key corresponds to all of the plain text after
the fasta format marker "``>``" that marks a new sequence.


Using Templates
~~~~~~~~~~~~~~~

tfasta will parse fasta files according to one of several templates
that are written to the conventions of several common fasta
file sources, like swissprot, pdb, nr (the non-redundant database),
and nrblast.

The latter example implicitly uses a the default template
called "``T_DEF``", yielding
dictionaries with the keys ``name`` and ``sequence``.
The ``sequence`` key is universal to all templates.

A complet list of templates (along with their keys) provided by
tfasta are:
  
    - ``T_DEF`` - plain old fasta line
        - ``name`` - everything after the ">"
    - ``T_SWISS`` - fasta files from swissprot
        - ``gi_num`` - between first set of "|"s
        - ``accession`` - between 3rd and 4th "|"
        - ``description`` - after last "|"
    - ``T_PDB`` - the fasta file of the entire pdb
        - ``idCode`` - first four characters after ">"
        - ``chainID`` - any non-whitespace characters after first "_"
        - ``type`` - non-whitespace immediately following first ":"
        - ``numRes`` - numbers immediatly following first ":"
        - ``description`` - stripped characters after numRes
    - ``T_NR`` - the protein non-redundant database
        - ``gi`` - between first set of "|"s
        - ``accession`` - between 3rd and 4th "|"
        - ``description`` - stripped characters before brackets
        - ``source`` - stripped characters inside brackets
    - ``T_NRBLAST`` - fasta file produced from blast output of the nr
        - ``gi`` - between first set of "|"s
        - ``accession`` - between 3rd and 4th "|"


An example using the ``T_NR`` template follows.
The nr fasta database has records that look like this::

    >gi|5524211|gb|AAD44166.1| cytochrome b [Elephas maximus maximus]
    LCLYTHIGRNIYYGSYLYSETWNTGIMLLLITMATAFMGYVLPWGQMSFWGATVITNLFSAIPYIGTNLV
    EWIWGGFSVDKATLNRFFAFHFILPFTMVALAGVHLTFLHETGSNNPLGLTSDSDKIPFHPYYTIKDFLG
    LLILILLLLLLALLSPDMLGDPDNHMPADPLNTPLHIKPEWYFLFAYAILRSVPNKLGGVLALFLSIVIL
    GLMPFLHTSKHRSMMLRPLSQALFWTLTMDLLTLTWIGSQPVEYPYTIIGQMASILYFSIILAFLPIAGX
    IENY


This file, named "``cytb.fas``", may be parsed using the
tfasta ``T_NR`` template::

    >>> from tfasta import fasta_parser
    >>> fast = fasta_parser("cytb.fas", template=T_NR)
    >>> f = fast.next()
    >>> print f['gi']
    5524211
    >>> print f['accession']
    AAD44166.1
    >>> print f['description']
    cytochrome b 
    >>> print f['source']
    Elephas maximus maximus
    >>> print f['sequence'][:60]
    LCLYTHIGRNIYYGSYLYSETWNTGIMLLLITMATAFMGYVLPWGQMSFWGATVITNLFS


Reading Small Fasta Files
~~~~~~~~~~~~~~~~~~~~~~~~~

Some fasta files may be several gigabytes in size. For this reason,
*fasta_parser()* reads fasta files incrementally by default,
such that only one sequence is read from the file at a time.

It is possible to force tfasta to read entire fasta files
at once by setting the ``greedy`` keyword to ``True``:

    fast = fasta_parser("cytb.fas", template=T_NR, greedy=True)

Greedy reading of files is better suited for situations where
the user does not want to keep the file fasta file open
for reading. It is not recommended for large files.


Preserving Gaps
~~~~~~~~~~~~~~~

Some sequences may have gaps ("-") such as those originating
from sequence alignments. To preserve gaps in the sequence 
set the ``dogaps`` keyword to ``True``::

    fast = fasta_parser("alignment.fas", dogaps=True)


Reading Fasta Strings
~~~~~~~~~~~~~~~~~~~~~

Not all fasta will originate from text files.
Often, it is necessary to parse a python ``str`` directly
using *string_fasta_parser()*::

    f = string_fasta_parser(astr)

For example::

    >>> from tfasta import string_fasta_parser
    >>> astr = """
               > Abeta in a string
               DAEFRHDSGYEVHHQKLVFFAEDVGSNKGAIIGLMVGGVVIA
               """
    >>> fast = string_fasta_parser(astr)
    >>> f = fast.next()
    >>> print f['name']
    Abeta in a string
    >>> print f['sequence']
    DAEFRHDSGYEVHHQKLVFFAEDVGSNKGAIIGLMVGGVVIA

The *string_fasta_parser()* function takes all of the
same arguments as *fasta_parser()* except for the ``greedy``
keyword argument.


Creating Fasta
~~~~~~~~~~~~~~

Fasta formatted text can be created one at a time or in bunches
with the functions *make_fasta()* and *make_fasta_from_dict()*.
The following example creates fasta text from a name
and unformatted sequence::

    >>> from tfasta import make_fasta
    >>> name = "OVAX_CHICK GENE X PROTEIN N-Term"
    >>> seq = """
    ...       QIKDLLVSSSTDLDTTLVLVNAIYFKGMWK
    ...       afnaedtrempfhvtkqeskpvqmmcmnnsfnvatlpae
    ...       KMKILELPFASGDLSMLV
    ...       """
    >>> print make_fasta(name, seq, width=50)
    >OVAX_CHICK GENE X PROTEIN N-Term
    QIKDLLVSSSTDLDTTLVLVNAIYFKGMWKAFNAEDTREMPFHVTKQESK
    PVQMMCMNNSFNVATLPAEKMKILELPFASGDLSMLV

The default width of a *make_fasta()* formatted
fasta sequence is 60. In this latter example, the
width is changed to 50 using the ``width`` keyword.

This next example creates fasta from an ordered dictionary
(available at https://pypi.python.org/pypi/ordereddict/
or built in to Python 2.7+)::

    >>> from tfasta import make_fasta_from_dict
    >>> from collections import OrderedDict  # python 2.7+
    >>> od = OrderedDict({ "First 10":  " 1 QIKDLLVSSS",
    ...                    "Second 10": "11 tdldttlvlv" })
    print make_fasta_from_dict(od)
    >First 10
    QIKDLLVSSS
    >Second 10
    TDLDTTLVLV

Using an ordered dictionary is not required, but it does
ensure control over the order of the sequences in the fasta.
Any well-behaved "mapping object", like the built-in ``dict``
will work.

Note that *make_fasta()* and *make_fasta_from_dict()*
both ignore all characters except letters and "-".


Creating Templates
~~~~~~~~~~~~~~~~~~

The creation of templates uses python regular expressions
to find fields in the first line of a sequence record
within a fasta file (the line that begins with ">").

Each field must be "saved" by the regular expression
by wrapping its sub-expression in parentheses. For
example the ``T_NRBLAST`` template regular expression is::

    ^>gi\|([^|]*)\|[^|]*\|([^|]*)\|.*$
           ~~~~~           ~~~~~ 

Here, the sub-expressions underscored by ``~`` characters
are saved by virtue of the surrounding parentheses.

The keys by which these saved fields are referred in the
dictionaries are given names, in the order that they
occur in the regular expression::

    regex = r'^>gi\|([^|]*)\|[^|]*\|([^|]*)\|.*$'
    fields = ("gi", "accession")

The next step is to create a template using the *FastaTemplate*
class::

    t_nrblast = FastaTemplate(regex, fields)

Here the template ``t_nrblast`` functions exactly as
the tfasta provided template ``T_NRBLAST``.

Finally, use the template to parse fasta::

    fasta_parser(file_name, template=t_nrblast)

The following examples shows templating put all together::

    >>> from tfasta import fasta_parser, FastaTemplate
    >>> regex = r'^>gi\|([^|]*)\|[^|]*\|([^|]*)\|.*$'
    >>> fields = ("gi", "accession")
    >>> t_nrblast = FastaTemplate(regex, fields)
    >>> fast = fasta_parser("short-extended.fas", template=t_nrblast)
    >>> f = fast.next()
    >>> print f['accession']
    ZP_00133915.1
    >>> print f['sequence'][:60]
    ATGQVIGTFTVRNDNGLHARPSAVLVQTLKPFAAKVTVENLDRGTAPANAKSTMKVVALG


.. _`pip`: http://www.pip-installer.org/en/latest/
.. _`easy_install`: http://peak.telecommunity.com/DevCenter/EasyInstall

.. toctree::
   :maxdepth: 2
   :numbered:


Indices and tables
==================

* :ref:`genindex`
* :ref:`modindex`
* :ref:`search`
