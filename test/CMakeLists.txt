itk_module_test()

set(StatismoITKTests
  itkStandardImageRepresenterTest.cxx
  itkStandardMeshRepresenterTest.cxx
  )

CreateTestDriver(StatismoITK "${StatismoITK_LIBRARIES};${StatismoITK-Test_LIBRARIES}" "${StatismoITKTests}")

itk_add_test( NAME itkStandardImageRepresenterTest
  COMMAND StatismoITKTestDriver itkStandardImageRepresenterTest
    DATA{Input/hand_images/hand-1.vtk}
    DATA{Input/hand_images/hand-2.vtk}
    DATA{Input/hand_dfs/df-hand-1.vtk}
    DATA{Input/hand_dfs/df-hand-2.vtk}
)
itk_add_test( NAME itkStandardMeshRepresenterTest
  COMMAND StatismoITKTestDriver itkStandardMeshRepresenterTest
    DATA{Input/hand_polydata/hand-0.vtk}
    DATA{Input/hand_polydata/hand-1.vtk}
)
