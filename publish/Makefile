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
	@cd ap; make clean
	@cd docker; make clean
	@cd kubernetes; make clean
