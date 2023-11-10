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

# Find the gomplate program. ###################################################
find_program(GOMPLATE gomplate)
message(STATUS "Found gomplate: ${GOMPLATE}")

# Find the yq program. #########################################################
find_program(YQ yq)
message(STATUS "Found yq: ${YQ}")

# Render the LaTeX template. ###################################################
add_custom_target(
  render
  COMMAND
    ${GOMPLATE} -c .=${PROJECT_SOURCE_DIR}/src/cv.yaml -f
    ${PROJECT_SOURCE_DIR}/src/template.tex -o
    ${PROJECT_BINARY_DIR}/src/template.tex > gomplate.log
  BYPRODUCTS ${PROJECT_BINARY_DIR}/src/template.tex
  WORKING_DIRECTORY ${PROJECT_BINARY_DIR}
  COMMENT "Render the template.tex")

# Convert cv.yaml to cv.json. ##################################################
add_custom_target(
  convert
  COMMAND ${YQ} -o=json eval ${PROJECT_SOURCE_DIR}/src/cv.yaml >
          ${PROJECT_BINARY_DIR}/src/cv.json
  BYPRODUCTS ${PROJECT_BINARY_DIR}/src/cv.json
  WORKING_DIRECTORY ${PROJECT_BINARY_DIR}
  COMMENT "Convert cv.yaml to cv.json")

# Compile the LaTeX document. ##################################################
add_latex_document(
  src/cv.tex
  INPUTS
  src/cv.yaml
  src/template.tex
  fonts/Fontin.otf
  fonts/Fontin-SmallCaps.otf
  fonts/Fontin-Italic.otf
  fonts/Fontin-Bold.otf
  BIBFILES
  src/jvanhare.bib
  DEPENDS
  convert
  render
  USE_BIBLATEX)

# Install cv.pdf. ##############################################################
install(FILES ${PROJECT_BINARY_DIR}/cv.pdf ${PROJECT_BINARY_DIR}/src/cv.json
        DESTINATION ${PROJECT_SOURCE_DIR})
