# -- start notice --
# THIS PROGRAM REQUIRED A SCALE AND LCD TO BE CONNECTED TO THE HUB
# THIS PROGRAM REQUIRES Phidget22 TO BE INSTALLED
# --  end notice  --

from Phidget22.Devices.VoltageRatioInput import *
from Phidget22.Devices.LCD import *
from time import sleep

# initialize lcd, scale, and data variables
lcd = LCD()
scale = VoltageRatioInput()
data = []

# attach lcd and scale
lcd.openWaitForAttachment(1000)
scale.openWaitForAttachment(1000)

# initial display state
lcd.setBacklight(0.2)
lcd.writeText(LCDFont.FONT_5x8, 20, 12, "Finding Offset...")
lcd.writeText(LCDFont.FONT_5x8, 30, 26, "DO NOT TOUCH")
lcd.flush()


# calculate offset of scale
def find_offset():
    offsets = []
    # find average force on scale without anyone touching
    for i in range(0, 10):
        offsets.append(scale.getVoltageRatio() * 46107)
        sleep(scale.getDataInterval() / 1000)
    return int(round(sum(offsets) / len(offsets)))


# calculate y location given a force and maximum y-value
def find_pixel(current_value, current_max):
    #
    pixel = 63 - int(round((current_value - offset) * 63 / current_max))
    # in case offset is slightly too large, don't allow negative y values (forces)
    if pixel < 0:
        pixel = 0
    return pixel


offset = find_offset()


def update_graph():
    # find current force
    force = scale.getVoltageRatio() * 46107
    # reset lcd
    lcd.clear()
    # insert current force at beginning of array
    data.insert(0, force)
    current_data = data
    current_max = 10
    # make set of data only the most recent 34 forces
    if len(data) >= 34:
        current_data = data[0:34]
        # calculate maximum y value of graph
        for i in range(0, 34):
            if current_data[i] - offset > current_max:
                current_max = round(current_data[i]) - offset
    else:
        # calculate maximum y value of graph
        for i in range(0, len(current_data)):
            if current_data[i] - offset > current_max:
                current_max = round(current_data[i]) + 10 - offset
    # loop over all forces in order to draw them on graph
    for i in range(0, len(current_data)):
        # in case offset is slightly too large, don't allow negative y values (forces)
        if current_data[i] - offset < 0:
            current_data[i] = offset
        pixel = find_pixel(current_data[i], current_max)
        lcd.drawPixel(i*3 + len(str(current_max)) * 3 + 10, pixel, True)
        # draw line between current and previous point if it is not the first point drawn
        # (because it has no previous points)
        if not i == 0:
            x1 = (i-1) * 3 + len(str(current_max)) * 3 + 10
            x2 = i*3 + len(str(current_max)) * 3 + 10
            y1 = find_pixel(current_data[i-1], current_max)
            y2 = pixel
            lcd.drawLine(x1, y1, x2, y2)
    # write maximum y value
    lcd.writeText(LCDFont.FONT_5x8, 0, 0, str(current_max))
    # draw y axis
    lcd.drawLine(len(str(current_max)) * 5 + 5, 0, len(str(current_max)) * 5 + 5, 63)
    lcd.flush()


delay = scale.getDataInterval()
while True:
    update_graph()
    sleep(delay / 1000)
