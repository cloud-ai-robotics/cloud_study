.PHONY: all clean

all: cloudy.tex ap.tex kubernetes.tex docker.tex References.bib
	@pdflatex cloudy.tex
	@bibtex cloudy
	@pdflatex ap.tex
	@pdflatex docker.tex
	@pdflatex kubernetes.tex
	@pdflatex cloudy.tex
	@pdflatex cloudy.tex

clean:
	@rm *.aux *.blg *.bbl *.log *.pdf
