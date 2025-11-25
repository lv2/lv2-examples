# Copyright 2021-2025 David Robillard <d@drobilla.net>
# SPDX-License-Identifier: 0BSD OR ISC

project(
  'lv2-examples',
  ['c'],
  default_options: [
    'b_ndebug=if-release',
    'buildtype=release',
    'c_std=c99',
  ],
  license: 'ISC',
  meson_version: '>= 0.56.0',
  version: '0.0.1',
)

lv2_examples_source_root = meson.current_source_dir()
lv2_examples_build_root = meson.current_build_dir()

#######################
# Compilers and Flags #
#######################

# Required tools
pkg = import('pkgconfig')
pymod = import('python')
cc = meson.get_compiler('c')

# Set global warning suppressions
warning_level = get_option('warning_level')
c_suppressions = []
if cc.get_id() in ['clang', 'emscripten']
  if warning_level == 'everything'
    c_suppressions += [
      '-Wno-bad-function-cast',
      '-Wno-cast-align',
      '-Wno-cast-function-type-strict',
      '-Wno-declaration-after-statement',
      '-Wno-double-promotion',
      '-Wno-float-conversion',
      '-Wno-implicit-float-conversion',
      '-Wno-padded',
      '-Wno-shorten-64-to-32',
      '-Wno-sign-conversion',
      '-Wno-switch-default',
      '-Wno-switch-enum',
      '-Wno-unsafe-buffer-usage',
    ]

    if not meson.is_cross_build()
      c_suppressions += ['-Wno-poison-system-directories']
    endif
  endif

  c_suppressions += [
    '-Wno-unused-parameter',
  ]

elif cc.get_id() == 'gcc'
  if warning_level == 'everything'
    c_suppressions += [
      '-Wno-bad-function-cast',
      '-Wno-cast-align',
      '-Wno-conversion',
      '-Wno-double-promotion',
      '-Wno-float-equal',
      '-Wno-inline',
      '-Wno-padded',
      '-Wno-suggest-attribute=pure',
      '-Wno-switch-default',
      '-Wno-switch-enum',
      '-Wno-unsuffixed-float-constants',
    ]
  endif

  c_suppressions += [
    '-Wno-unused-parameter',
  ]

elif cc.get_id() == 'msvc'
  c_suppressions += [
    '/wd4061', # enumerator in switch is not explicitly handled
    '/wd4100', # unreferenced parameter
    '/wd4244', # conversion with possible loss of data
    '/wd4267', # conversion from size_t to a smaller type
    '/wd4310', # cast truncates constant value
    '/wd4365', # signed/unsigned mismatch
    '/wd4464', # relative include path contains ".."
    '/wd4514', # unreferenced inline function has been removed
    '/wd4706', # assignment within conditional expression
    '/wd4710', # function not inlined
    '/wd4711', # function selected for automatic inline expansion
    '/wd4820', # padding added after construct
    '/wd5045', # will insert Spectre mitigation for memory load
  ]
endif

c_suppressions = cc.get_supported_arguments(c_suppressions)

##################
# LV2 Dependency #
##################

# Only fall back to local subproject if this is the top project
# (Workaround for older meson override bug)
lv2_fallback = []
if not meson.is_subproject()
  lv2_fallback = ['lv2', 'lv2_dep']
endif

# Use the system LV2, or a stripped down local subproject
lv2_dep = dependency(
  'lv2',
  default_options: [
    'bundles=false',
    'docs=disabled',
    'headers=false',
    'lint=false',
    'lv2dir=' + get_option('lv2dir'),
    'old_headers=false',
    'online_docs=false',
    'tests=disabled',
    'tools=disabled',
  ],
  fallback: lv2_fallback,
  include_type: 'system',
  version: '>= 1.18.11',
)

# Install to the LV2 directory from the dependency by default
lv2dir = get_option('lv2dir')
if lv2dir == ''
  lv2dir = lv2_dep.get_variable(pkgconfig: 'lv2dir', internal: 'lv2dir')
endif

########
# Lint #
########

python_scripts = files('plugins/literasc.py')

if get_option('lint')
  # Check release metadata
  if not meson.is_subproject()
    autoship = find_program('autoship', required: false)
    if autoship.found()
      autoship_args = ['test', lv2_examples_source_root]
      test('autoship', autoship, args: autoship_args, suite: 'data')
    endif
  endif

  # Check licensing metadata
  reuse = find_program('reuse', required: false)
  if reuse.found()
    reuse_args = ['--root', lv2_examples_source_root, 'lint']
    test('REUSE', reuse, args: reuse_args, suite: 'data')
  endif

  # Check script formatting
  black = find_program('black', required: get_option('tests'))
  if black.found()
    black_opts = ['-l', '79', '-q', '--check']
    test('black', black, args: black_opts + python_scripts, suite: 'scripts')
  endif

  # Check scripts for errors with flake8
  flake8 = find_program('flake8', required: get_option('tests'))
  if flake8.found()
    test('flake8', flake8, args: python_scripts, suite: 'scripts')
  endif

  # Check scripts for errors with pylint
  pylint = find_program('pylint', required: get_option('tests'))
  if pylint.found()
    test('pylint', pylint, args: python_scripts, suite: 'scripts')
  endif
endif

###########
# Bundles #
###########

subdir('lv2-examples.lv2')
subdir('plugins')

###########
# Project #
###########

if not meson.is_subproject()
  summary(
    {
      'Plugins': not get_option('plugins').disabled(),
      'Book': asciidoc.found(),
    },
    bool_yn: true,
    section: 'Components',
  )

  summary(
    {
      'Install prefix': get_option('prefix'),
      'LV2 bundles': lv2dir,
    },
    section: 'Directories',
  )
endif
