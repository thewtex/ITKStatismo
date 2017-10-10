set(Statismo_SRCS
  itkMinimalStandardRandomVariateGenerator.cxx
  )

add_library(Statismo ${Statismo_SRCS})
itk_module_link_dependencies()
itk_module_target(Statismo)
