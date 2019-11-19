.PHONY: all clean

all: cloudy.tex ap.tex kubernetes.tex docker.tex References.bib
	@pdflatex cloudy.tex
	@bibtex cloudy
	@pdflatex cloudy.tex

clean:
	@rm *.aux *.blg *.bbl *.log *.pdf
