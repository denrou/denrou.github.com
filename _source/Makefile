# You must have jekyll and R installed, and they must be in PATH. For R, you
# need to install the servr package.

all:
	Rscript -e "servr::jekyll(dir = '..', input = c('.', list.dirs('_source')), output = c('.', rep('_posts', length(list.dirs('_source')))))"

clean:
	rm -r ../_site
