OUT_DIR ?= build

LATEXMK_FLAGS ?= -pdf

TEX_SRCS := $(shell find . -type f -name "*.tex")

SRCS := $(TEX_SRCS)

pdf: $(OUT_DIR)/apresentacao.pdf

clean:
	@rm -rf $(OUT_DIR)

$(OUT_DIR)/apresentacao.pdf: $(SRCS)
	@rm -rf $(OUT_DIR)
	@latexmk $(LATEXMK_FLAGS) -output-directory=$(OUT_DIR) apresentacao.tex

.PHONY: pdf clean
