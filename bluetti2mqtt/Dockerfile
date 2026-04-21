ARG BUILD_FROM
FROM $BUILD_FROM

# Copy data for add-on
COPY rootfs /

# Install fork bluetti_mqtt version
RUN apk add --no-cache git \
    && pip install --no-cache-dir git+https://github.com/semitop7/bluetti_mqtt.git \
    && chmod a+x /run.sh

# Apply grid_charge_amps patch to ALL devices
RUN for device in ac500 ac300 ac200L ac60 ep500 ep500p ep600 eb3a; do \
    sed -i "s|self.struct.add_uint_field('battery_range_end'|self.struct.add_uint_field('grid_charge_amps', 3019)\n        self.struct.add_uint_field('battery_range_end'|" \
    /usr/local/lib/python3.14/site-packages/bluetti_mqtt/core/devices/$device.py 2>/dev/null || true; \
  done

# Apply dc_input_power2 and dc_input_voltage2 patch to AC500
RUN sed -i "s|self.struct.add_decimal_field('internal_dc_input_current', 88, 1)|self.struct.add_decimal_field('internal_dc_input_current', 88, 1)\n        self.struct.add_decimal_field('dc_input_voltage2', 170, 1)\n        self.struct.add_uint_field('dc_input_power2', 172)|" \
    /usr/local/lib/python3.14/site-packages/bluetti_mqtt/core/devices/ac500.py

# Add dc_input polling to AC500
RUN sed -i "s|ReadHoldingRegisters(70, 21),|ReadHoldingRegisters(70, 21),\n            ReadHoldingRegisters(170, 3),|" \
    /usr/local/lib/python3.14/site-packages/bluetti_mqtt/core/devices/ac500.py

# Create Python script to add grid_charge_amps and dc_input to MQTT config
RUN cat > /add_mqtt_fields.py << 'PYSCRIPT'
import sys
filepath = '/usr/local/lib/python3.14/site-packages/bluetti_mqtt/mqtt_client.py'
with open(filepath, 'r') as f:
    lines = f.readlines()

new_lines = []
i = 0
while i < len(lines):
    new_lines.append(lines[i])
    
    # Add grid_charge_amps after battery_range_end
    if "'battery_range_end': MqttFieldConfig(" in lines[i]:
        j = i + 1
        while j < len(lines) and ")," not in lines[j]:
            new_lines.append(lines[j])
            j += 1
        new_lines.append(lines[j])
        new_lines.append("    'grid_charge_amps': MqttFieldConfig(\n")
        new_lines.append("        type=MqttFieldType.NUMERIC,\n")
        new_lines.append("        setter=True,\n")
        new_lines.append("        advanced=False,\n")
        new_lines.append("        home_assistant_extra={\n")
        new_lines.append("            'name': 'Grid Charge Amps',\n")
        new_lines.append("            'step': 1,\n")
        new_lines.append("            'min': 0,\n")
        new_lines.append("            'max': 10,\n")
        new_lines.append("            'unit_of_measurement': 'A',\n")
        new_lines.append("        }\n")
        new_lines.append("    ),\n")
        i = j + 1
    
    # Add dc_input_power2 and dc_input_voltage2 after dc_input_current1
    elif "'dc_input_current1': MqttFieldConfig(" in lines[i]:
        j = i + 1
        while j < len(lines) and ")," not in lines[j]:
            new_lines.append(lines[j])
            j += 1
        new_lines.append(lines[j])
        new_lines.append("    'dc_input_voltage2': MqttFieldConfig(\n")
        new_lines.append("        type=MqttFieldType.NUMERIC,\n")
        new_lines.append("        setter=False,\n")
        new_lines.append("        advanced=False,\n")
        new_lines.append("        home_assistant_extra={\n")
        new_lines.append("            'name': 'DC Input Voltage 2',\n")
        new_lines.append("            'unit_of_measurement': 'V',\n")
        new_lines.append("            'device_class': 'voltage',\n")
        new_lines.append("            'state_class': 'measurement',\n")
        new_lines.append("            'force_update': True,\n")
        new_lines.append("        }\n")
        new_lines.append("    ),\n")
        new_lines.append("    'dc_input_power2': MqttFieldConfig(\n")
        new_lines.append("        type=MqttFieldType.NUMERIC,\n")
        new_lines.append("        setter=False,\n")
        new_lines.append("        advanced=False,\n")
        new_lines.append("        home_assistant_extra={\n")
        new_lines.append("            'name': 'DC Input Power 2',\n")
        new_lines.append("            'unit_of_measurement': 'W',\n")
        new_lines.append("            'device_class': 'power',\n")
        new_lines.append("            'state_class': 'measurement',\n")
        new_lines.append("            'force_update': True,\n")
        new_lines.append("        }\n")
        new_lines.append("    ),\n")
        i = j + 1
    else:
        i += 1

with open(filepath, 'w') as f:
    f.writelines(new_lines)
PYSCRIPT

RUN python3 /add_mqtt_fields.py

# Create Python script to publish dc_input_power2 and dc_input_voltage2
RUN cat > /add_mqtt_publish.py << 'PYSCRIPT'
import sys
filepath = '/usr/local/lib/python3.14/site-packages/bluetti_mqtt/mqtt_client.py'
with open(filepath, 'r') as f:
    content = f.read()

# Cerca la sezione dove pubblica dc_input_power1
search_str = """        if 'internal_dc_input_power' in msg.parsed:
            await client.publish(
                topic_prefix + 'dc_input_power1',
                payload=str(msg.parsed['internal_dc_input_power']).encode()
            )"""

replace_str = """        if 'internal_dc_input_power' in msg.parsed:
            await client.publish(
                topic_prefix + 'dc_input_power1',
                payload=str(msg.parsed['internal_dc_input_power']).encode()
            )
        if 'dc_input_power2' in msg.parsed:
            await client.publish(
                topic_prefix + 'dc_input_power2',
                payload=str(msg.parsed['dc_input_power2']).encode()
            )"""

if search_str in content:
    content = content.replace(search_str, replace_str)
    print("✓ Aggiunto publish dc_input_power2")

# Cerca la sezione dove pubblica dc_input_voltage1
search_str2 = """        if 'internal_dc_input_voltage' in msg.parsed:
            await client.publish(
                topic_prefix + 'dc_input_voltage1',
                payload=str(msg.parsed['internal_dc_input_voltage']).encode()
            )"""

replace_str2 = """        if 'internal_dc_input_voltage' in msg.parsed:
            await client.publish(
                topic_prefix + 'dc_input_voltage1',
                payload=str(msg.parsed['internal_dc_input_voltage']).encode()
            )
        if 'dc_input_voltage2' in msg.parsed:
            await client.publish(
                topic_prefix + 'dc_input_voltage2',
                payload=str(msg.parsed['dc_input_voltage2']).encode()
            )"""

if search_str2 in content:
    content = content.replace(search_str2, replace_str2)
    print("✓ Aggiunto publish dc_input_voltage2")

with open(filepath, 'w') as f:
    f.write(content)
PYSCRIPT

RUN python3 /add_mqtt_publish.py

CMD [ "/run.sh" ]