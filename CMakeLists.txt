find_package(Gettext REQUIRED)
find_program(XGETTEXT xgettext REQUIRED)

# Generate an up-to-date translation source .pot file from the C sources.
# NOTE: this is not your average build-something rule, this updates the
# rpm.pot SOURCE, which you then should commit into git.
add_custom_target(update-pot
	  COMMENT "Updating translation source"
	  COMMAND ${XGETTEXT}
			--package-name=${CMAKE_PROJECT_NAME}
			--package-version=${CMAKE_PROJECT_VERSION}
			--output=${CMAKE_CURRENT_SOURCE_DIR}/rpm.pot
			--language=C
			--no-location
			--keyword=_ --keyword=N_
			--files-from=${CMAKE_CURRENT_SOURCE_DIR}/POTFILES.in
	  WORKING_DIRECTORY ${CMAKE_SOURCE_DIR}
	  SOURCES ${POT_SOURCES}
)

# Discover our translations
file(READ LINGUAS linguas)
string(REPLACE " " ";" linguas ${linguas})
set(linguas ${linguas})
# Copy translations to build tree to avoid modifying source
foreach(lang ${linguas})
	string(STRIP ${lang} lang)
	set(po ${lang}.po)
	file(COPY ${po} DESTINATION ${CMAKE_CURRENT_BINARY_DIR})
	list(APPEND pos ${CMAKE_CURRENT_BINARY_DIR}/${po})
endforeach()
GETTEXT_CREATE_TRANSLATIONS(rpm.pot ALL ${pos})
