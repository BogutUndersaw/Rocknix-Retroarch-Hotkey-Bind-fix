# Rocknix-Retroarch-Hotkey-Bind-fix
This is a set of steps to edit the script in Rocknix that edits the hotkeys in Retroarch when you boot a game.


    1.) First ssh into Rocknix username is root pw is typically rocknix
    2.) run: cp /usr/bin/setsettings.sh /storage/scripts
    3.) run: nano /storage/scripts/setsettings.sh
    4.) delete the following: 
  
### Configure retroarch hotkeys
function configure_hotkeys() {
    log "Configure hotkeys..."
    local MY_CONTROLLER=$(grep -b4 js0 /proc/bus/input/devices | awk 'BEGIN {FS="\""}; /Name/ {printf $2}')

    ### Remove any input settings retroarch may have added.
    sed -i '/input_player[0-9]/d' ${RETROARCH_CONFIG}

    if [ "$(get_setting system.autohotkeys)" == "1" ]
    then
        if [ -e "/tmp/joypads/${MY_CONTROLLER}.cfg" ]
        then
            cp /tmp/joypads/"${MY_CONTROLLER}.cfg" /tmp
            sed -i "s# = #=#g" /tmp/"${MY_CONTROLLER}.cfg"
            source /tmp/"${MY_CONTROLLER}.cfg"
            for HKEYSETTING in input_enable_hotkey_btn input_bind_hold            \
                               input_exit_emulator_btn input_fps_toggle_btn       \
                               input_menu_toggle_btn input_save_state_btn         \
                               input_load_state_btn input_toggle_fast_forward_btn \
                               input_toggle_fast_forward_axis input_rewind_axis   \
                               input_rewind_btn
            do
                clear_setting "${HKEYSETTING}"
            done
            flush_settings
            if [ -z ${input_enable_hotkey_btn+x} ]
            then
                echo 'input_enable_hotkey_btn = '\"${input_select_btn}\" >>${RETROARCH_CONFIG}
            else
                echo 'input_enable_hotkey_btn = '\"${input_enable_hotkey_btn}\" >>${RETROARCH_CONFIG}
            fi
            cat <<EOF >>${RETROARCH_CONFIG}
input_bind_hold = "${input_select_btn}"
input_exit_emulator_btn = "${input_start_btn}"
input_fps_toggle_btn = "${input_y_btn}"
input_menu_toggle_btn = "${input_x_btn}"
input_save_state_btn = "${input_r_btn}"
input_load_state_btn = "${input_l_btn}"
EOF
            if [ -n "${input_r2_btn}" ] && \
               [ -n "${input_l2_btn}" ]
            then
                cat <<EOF >>${RETROARCH_CONFIG}
input_toggle_fast_forward_axis = "nul"
input_toggle_fast_forward_btn = "${input_r2_btn}"
input_rewind_axis = "nul"
input_rewind_btn = "${input_l2_btn}"
EOF
            elif [ -n "${input_r2_axis}" ] && \
                 [ -n "${input_l2_axis}" ]
            then
                cat <<EOF >>${RETROARCH_CONFIG}
input_toggle_fast_forward_axis = "${input_r2_axis}"
input_toggle_fast_forward_btn = "nul"
input_rewind_axis = "${input_l2_axis}"
input_rewind_btn = "nul"
EOF
            fi
            rm -f /tmp/"${MY_CONTROLLER}.cfg"
        fi
    fi
}

  5.) run: chmod +x /storage/scripts/setsettings.sh
  6.) run: mount -o bind /storage/scripts/setsettings.sh /usr/bin/setsettings.sh
