# Curriculum Vitæ. #############################################################
cmake_minimum_required(VERSION 3.21)
project(cv)
list(APPEND CMAKE_MODULE_PATH ${CMAKE_CURRENT_SOURCE_DIR}/cmake)
set(PDFLATEX_COMPILER xelatex)
include(UseLATEX)

# Download on GitHub my bibtex file. ###########################################
file(
  DOWNLOAD
  https://raw.githubusercontent.com/jvanhare/bibliography/master/src/jvanhare.bib
  ${PROJECT_BINARY_DIR}/src/jvanhare.bib)

# Find the Horus program. ######################################################
find_program(HORUS horus)
message(STATUS "Found Horus: ${HORUS}")

# Render the LaTeX template. ###################################################
add_custom_target(
  render
  COMMAND
    ${HORUS} render -t ${PROJECT_SOURCE_DIR}/src/template.tex -y
    ${PROJECT_SOURCE_DIR}/src/cv.yml -o ${PROJECT_BINARY_DIR}/src/template.tex
    --latex > horus.log
  BYPRODUCTS ${PROJECT_BINARY_DIR}/src/template.tex
  WORKING_DIRECTORY ${PROJECT_BINARY_DIR}
  COMMENT "Render the template.tex")

# Compile the LaTeX document. ##################################################
add_latex_document(
  src/cv.tex
  INPUTS
  src/cv.yml
  src/template.tex
  fonts/Fontin.otf
  fonts/Fontin-SmallCaps.otf
  fonts/Fontin-Italic.otf
  fonts/Fontin-Bold.otf
  BIBFILES
  src/jvanhare.bib
  DEPENDS
  render
  USE_BIBLATEX)

# Install cv.pdf. ##############################################################
install(FILES ${PROJECT_BINARY_DIR}/cv.pdf DESTINATION ${PROJECT_SOURCE_DIR})
