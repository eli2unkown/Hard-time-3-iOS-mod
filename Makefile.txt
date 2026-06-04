DEBUG = 0
FINALPACKAGE = 1
THEOS_DEVICE_IP = localhost

TARGET := iphone:clang:latest:14.0
ARCHS = arm64 arm64e

include $(THEOS)/makefiles/common.mk

TWEAK_NAME = HardTime3ModMenu
HardTime3ModMenu_FILES = Tweak.xm
HardTime3ModMenu_FRAMEWORKS = UIKit CoreGraphics

include $(THEOS_MAKE_PATH)/tweak.mk
