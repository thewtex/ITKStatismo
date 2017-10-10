itk_module_test()

set(StatismoTests
  itkStandardImageRepresenterTest.cxx
  itkStandardMeshRepresenterTest.cxx
  )

CreateTestDriver(Statismo "${Statismo_LIBRARIES}" "${StatismoTests}")

itk_add_test( NAME itkStandardImageRepresenterTest
  COMMAND StatismoTestDriver itkStandardImageRepresenterTest ${statismo_SOURCE_DIR}/data
)
itk_add_test( NAME itkStandardMeshRepresenterTest
  COMMAND StatismoTestDriver itkStandardMeshRepresenterTest ${statismo_SOURCE_DIR}/data
)
