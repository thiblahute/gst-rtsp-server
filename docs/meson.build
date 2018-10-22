html_theme = 'https://github.com/hotdoc/hotdoc_lumen_theme/releases/download/0.6/hotdoc_lumen_theme-0.6.tar.xz?sha256=0e2f175f4cf8c00ed7ac5014e30c806a294b0d3818565eb74e1424a948e8a452'
build_hotdoc = false

hotdoc_extension_dir = []
required_hotdoc_extensions = ['gi-extension']
if gst_dep.type_name() == 'internal'
    gst_proj = subproject('gstreamer')
    plugins_cache_generator = gst_proj.get_variable('plugins_cache_generator')
    if gst_proj.get_variable('build_hotdoc')
        hotdoc_extension_dir = [gst_proj.get_variable('hotdoc_extension_dir')]
    else
        required_hotdoc_extensions += ['gst-extension']
    endif
else
    required_hotdoc_extensions += ['gst-extension']
    plugins_cache_generator = find_program(join_paths(gst_dep.get_pkg_variable('libexecdir'), 'gstreamer-' + api_version, 'gst-plugins-doc-cache-generator'),
        required: false)
endif

plugins_cache = join_paths(meson.current_source_dir(), 'gst_plugins_cache.json')
if plugins_cache_generator.found()
    plugins_doc_dep = custom_target('rtsp-server-plugins-doc-cache',
        command: [plugins_cache_generator, plugins_cache, '@INPUT@'],
        input: plugins,
        output: 'gst_plugins_cache.json',
    )
else
    warning('GStreamer plugin inspector for documentation not found, can\'t update the cache')
endif

hotdoc_p = find_program('hotdoc', required: get_option('doc'))
if not hotdoc_p.found()
    message('Hotdoc not found, not building the documentation')
    subdir_done()
endif

hotdoc = import('hotdoc')
if not hotdoc.has_extensions(required_hotdoc_extensions)
    if get_option('doc').enabled()
        error('Documentation enabled but gi-extension missing')
    endif

    message('@0@ extensions not found, not building documentation'.format(required_hotdoc_extensions))
    subdir_done()
endif

if not build_gir
    if get_option('doc').enabled()
        error('Documentation enabled but introspection not built.')
    endif

    message('Introspection not built, can\'t build the documentation')
    subdir_done()
endif

build_hotdoc = true
hotdoc = import('hotdoc')

libs_doc = [hotdoc.generate_doc('gst-rtsp-server',
    project_version: api_version,
    gi_c_sources: ['../gst/rtsp-server/*.[hc]'],
    gi_sources: rtsp_server_gir[0].full_path(),
    sitemap: 'sitemap.txt',
    index: 'index.md',
    gi_index: 'index.md',
    gi_smart_index: true,
    gi_order_generated_subpages: true,
    html_theme: html_theme,
)]

plugins_doc = [hotdoc.generate_doc('rtspclientsink',
    project_version: api_version,
    sitemap: 'plugin-sitemap.txt',
    index: 'plugin-index.md',
    gst_index: 'plugin-index.md',
    gst_c_sources: ['../gst/rtsp-sink/*.[ch]'],
    gst_dl_sources: [rtspsink.full_path()],
    gst_smart_index: true,
    dependencies: gst_rtsp_server_deps + [rtspsink],
    extra_extensions_paths: hotdoc_extension_dir,
    gst_cache_file: plugins_cache,
    gst_plugin_name: 'rtspclientsink',
    html_theme: html_theme,
)]
doc = libs_doc[0]