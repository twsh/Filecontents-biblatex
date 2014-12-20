#!/usr/bin/env python3

import argparse
import os.path
import re
import subprocess

from bibtexparser.bparser import BibTexParser
from bibtexparser.bwriter import to_bibtex
from bibtexparser.customization import *


# bibtexparser customisations
def bibdesk(record):
    """
    Remove BibDesk fields.

    :param record: the record.
    :type record: dict
    :returns: dict -- the modified record.
    """
    if "read" in record:
        del record["read"]
    if "bdsk-file-1" in record:
        del record["bdsk-file-1"]
    if "date-added" in record:
        del record["date-added"]
    if "date-modified" in record:
        del record["date-modified"]
    return record


# For some reason URL fields are converted to 'link' fields
def link_to_url(record):
    """
    Turn 'link' fileds to 'url'.

    :param record: the record.
    :type record: dict
    :returns: dict -- the modified record.
    """
    if "link" in record and "url" not in record:
        record["url"] = record["link"]
        del record["link"]
    return record


def customizations(record):
    """Use some functions delivered by the library

    :param record: a record
    :returns: -- customized record
    """
    record = link_to_url(record)
    record = bibdesk(record)
    return record


if __name__ == "__main__":
    parser = argparse.ArgumentParser()
    parser.add_argument('target')
    parser.add_argument('--output', default='standalone.tex')
    parser.add_argument('--bcf')
    args = parser.parse_args()
    target = args.target
    filename = os.path.splitext(target)[0]
    # The bcf file to get the data from
    # Default to looking for one with the same name as the target
    if args.bcf:
        bcf = args.bcf
    else:
        bcf = '{}.bcf'.format(filename)
    tex = target
    output = args.output
    # The bibliography that biber will make
    if args.bcf:
        biber_bib = '{}_biber.bib'.format(os.path.splitext(args.bcf)[0])
    else:
        biber_bib = '{}_biber.bib'.format(filename)
    if os.path.isfile(bcf):
        subprocess.check_call(
            [
                'biber',
                '--output_format=bibtex',
                '--output_resolve',
                bcf
            ]
        )
    try:
        with open(biber_bib) as f:
            bibliography = BibTexParser(
                f.read(),
                customization=customizations,
                ignore_nonstandard_types=False
                # Otherwise bibtexparser will complain if I give it a collection
            )
            filecontents = (
                '\\begin{filecontents}{\\jobname.bib}\n'
                + to_bibtex(bibliography)
                + '\\end{filecontents}\n'
                + '\\addbibresource{\\jobname.bib}\n'
            )
        os.remove(biber_bib)
        with open(tex) as f:
            text = f.read()
        i = text.index('\\begin{document}')
        text = '{}{}{}'.format(
            re.sub(
                '\\\\addbibresource{.+}',
                '',
                text[:i]
            ),
            filecontents,
            text[i:]
        )
        with open(output, 'w') as f:
            f.write(text)
        print('I made a file called {}.'.format(output))
    except FileNotFoundError:
        print(
            "I couldn't make a bibliography, "
            "probably because I couldn't find a bcf file called {}.".format(bcf)
        )
