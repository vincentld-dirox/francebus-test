# Spec Delta

## Purpose

The plugin's WordPress settings page, where a site administrator sets which website the plugin serves (France Bus or Wedding), its language, and switches it on or off.

## ADDED Requirements

### Requirement: Settings page
The plugin SHALL have a settings page in the WordPress administration where an administrator sets the website it serves (France Bus or Wedding), its language (French or English) and an on/off switch.

#### Scenario: Settings saved
- **WHEN** an administrator sets the website to Wedding, the language to English and the switch to on, then saves
- **THEN** the settings page shows those values when opened again

### Requirement: Administrators only
Only WordPress administrators SHALL be able to see and change the plugin's settings.

#### Scenario: Editor cannot change the settings
- **WHEN** a WordPress user who is not an administrator tries to open the settings page
- **THEN** WordPress refuses access

### Requirement: Website setting
The website setting SHALL decide which website's style the estimate window uses.

#### Scenario: Wedding website set
- **GIVEN** the website setting is Wedding
- **WHEN** the estimate window opens
- **THEN** it uses the wedding site's style

### Requirement: On/off switch
With the switch off, the plugin SHALL add nothing to the form page and send nothing to the engine: the address fields work as plain text fields, there is no estimate window, and JetFormBuilder sends its e-mail exactly as today.

#### Scenario: Plugin switched off
- **GIVEN** the switch is off
- **WHEN** a visitor types an address and submits the form
- **THEN** no suggestion is shown, no estimate window opens, the engine receives nothing, and JetFormBuilder sends its e-mail as today
