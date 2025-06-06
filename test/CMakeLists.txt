#[[
Copyright (c) 2014-2024 Key4hep-Project.

This file is part of Key4hep.
See https://key4hep.github.io/key4hep-doc/ for further info.

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
]]
include(CTest)

set(DETECTOR $ENV{K4GEO}/FCCee/CLD/compact/CLD_o2_v07/CLD_o2_v07.xml)
set(CLDConfig_DIR ${CMAKE_CURRENT_LIST_DIR}/../CLDConfig)

add_test(NAME ddsim_lcio
         WORKING_DIRECTORY ${CLDConfig_DIR}
         COMMAND ddsim -S cld_steer.py -N 3 --inputFile ../test/yyxyev_000.stdhep --outputFile test.slcio --compactFile ${DETECTOR}
)

add_test(NAME ddsim_cld_arc
         WORKING_DIRECTORY ${CLDConfig_DIR}
         COMMAND ddsim -S cld_arc_steer.py -N 10 -G --gun.direction "1 0 0"
)

# FIXME: need to call k4run from CLDConfig dir where the pandora config lives
add_test(NAME lcio_input
         WORKING_DIRECTORY ${CLDConfig_DIR}
         COMMAND k4run --inputFiles=test.slcio --outputBasename=rec_test_lcio CLDReconstruction.py --GeoSvc.detectors=${DETECTOR}
)
set_property(TEST lcio_input APPEND PROPERTY DEPENDS ddsim_lcio)

add_test(NAME ddsim_edm4hep
         WORKING_DIRECTORY ${CLDConfig_DIR}
         COMMAND ddsim -S cld_steer.py -N 3 --inputFile ../test/yyxyev_000.stdhep --outputFile test.edm4hep.root --compactFile ${DETECTOR}
)

add_test(NAME edm4hep_input
         WORKING_DIRECTORY ${CLDConfig_DIR}
         COMMAND k4run --inputFiles=test.edm4hep.root --outputBasename=rec_test_edm4hep CLDReconstruction.py --GeoSvc.detectors=${DETECTOR}
)
set_property(TEST edm4hep_input APPEND PROPERTY DEPENDS ddsim_edm4hep)

add_test(NAME trkOnly
         WORKING_DIRECTORY ${CLDConfig_DIR}
         COMMAND k4run --trackingOnly --inputFiles=test.edm4hep.root --outputBasename=trkOnly_test_edm4hep CLDReconstruction.py --GeoSvc.detectors=${DETECTOR}
)
set_property(TEST trkOnly APPEND PROPERTY DEPENDS ddsim_edm4hep)

add_test(NAME LCFIJet
         WORKING_DIRECTORY ${CLDConfig_DIR}
         COMMAND k4run --enableLCFIJet --inputFiles=test.edm4hep.root --outputBasename=trkOnly_test_edm4hep CLDReconstruction.py --GeoSvc.detectors=${DETECTOR}
)
set_property(TEST LCFIJet APPEND PROPERTY DEPENDS ddsim_edm4hep)

add_test(NAME tracking_truth
         WORKING_DIRECTORY ${CLDConfig_DIR}
         COMMAND k4run --trackingOnly --truthTracking --inputFiles=test.edm4hep.root --outputBasename=trkOnly_truth_test_edm4hep CLDReconstruction.py --GeoSvc.detectors=${DETECTOR}
)
set_property(TEST tracking_truth APPEND PROPERTY DEPENDS ddsim_edm4hep)

add_test(NAME tracking_conformal
         WORKING_DIRECTORY ${CLDConfig_DIR}
         COMMAND k4run --trackingOnly --conformalTracking --inputFiles=test.edm4hep.root --outputBasename=trkOnly_conformal_test_edm4hep CLDReconstruction.py --GeoSvc.detectors=${DETECTOR}
)
set_property(TEST tracking_conformal APPEND PROPERTY DEPENDS ddsim_edm4hep)

add_test(NAME tracking_both
         WORKING_DIRECTORY ${CLDConfig_DIR}
         COMMAND k4run --trackingOnly --conformalTracking --truthTracking CLDReconstruction.py
)
set_property(TEST tracking_both APPEND PROPERTY PASS_REGULAR_EXPRESSION "error: argument --truthTracking: not allowed with argument --conformalTracking")

add_test(NAME Help
         WORKING_DIRECTORY ${CLDConfig_DIR}
         COMMAND k4run --help CLDReconstruction.py
)
set_property(TEST Help APPEND PROPERTY PASS_REGULAR_EXPRESSION "show this help message and exit")

add_test(NAME marlin
         WORKING_DIRECTORY ${CLDConfig_DIR}
         COMMAND Marlin --global.LCIOInputFiles=test.slcio --InitDD4hep.DD4hepXMLFile=${DETECTOR} CLDReconstruction.xml
)
set_property(TEST marlin APPEND PROPERTY DEPENDS ddsim_lcio)
