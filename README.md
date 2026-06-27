# Looking for RP

## Status

**Community Supported:** You can contact me (Trashcan) if you have issues with this code. I cannot guarantee timeline but I will respond.

This works with Ares 2.0+ as of time of release. **NOTE: The install directions are written for 2.13+. Older versions can still use this plugin but will require custom code configuration beyond the scope of the built-in custom hooks. Pre-2.13 Ares games should follow [these alternative steps](https://github.com/Red-Planet-MU/ares-lookingforrp-plugin/blob/main/README%20(pre%202_13).md) or upgrade to 2.13+ prior to installation.**

## Overview + Credits

This plugin adds a section to the web sidebar, the 'Play' screen, and the in-client 'who' list to indicate if a particular character is Looking for RP. The flag is set on a timer and can go up to 3 hours, and can be optionally set to indicate that the character is only looking for a 'Txt' scene if the [Text Messages plugin](https://github.com/spiritlake/ares-txt-plugin) is installed. Both emit to the RP Requests channel, although players can disable the announcement if they prefer. This work builds on code originally written by Tat for Shattered after the original idea was seen on Concordia.

## Web Portal

All functions work on the web if you follow the additional steps. This is easiest if you have a Github fork of your own but you can also edit the custom hooks directly in your web portal (yourgame.com/custom-code) and edit the web-portal files directly in your server shell.

## Installation

1. From a bit with the Coder role in the client., run `plugin/install <github url>`.
2. Run `ruby LookingForRp.install_setup`.
3. Edit `custom_who_fields.rb` (in `plugins/profile/public`) with the below. NOTE: `custom_who_fields` already exists. If you have custom data already, you will need to add this statement to your existing code.
```
def self.custom_who_field(char, field_type, value, width)
      when 'lookingforrp'
        looking_for_rp = char.looking_for_rp
        case char.looking_for_rp_type
          when "scene"
            flag = "%xgRP%xn"
          when "text"
            flag = "%xmTXT%xn"
        end
        return looking_for_rp ? flag.ljust(width+6) : ""
      else 
        return nil
      end

```
4. Edit `who.yml` in your game's config files to add (after Status, or wherever you prefer):
```
- field: lookingforrp
  width: 5
  title: RP?
```
5. Edit `custom_web_data.rb` (in `plugins/website`) with the below. NOTE: `custom_sidebar_data` already exists. If you have custom data already, you will need to add these fields to your existing data. 
```
    def self.custom_sidebar_data(viewer)
      return {
        lfrp_icons: LookingForRp.web_list,
        txt_extra_installed: Manage.is_extra_installed?("txt")
      }
    end
```
6. Edit `custom_scene_data.rb` (in `plugins/scenes`) with the below. NOTE: `custom_scene_data` already exists. If you have custom data already, you will need to add these fields to your existing data. 
```
    def self.custom_scene_data(viewer)
      return {
        lfrp_icons: LookingForRp.web_list,
        txt_extra_installed: Manage.is_extra_installed?("txt")
      }
    end
```
7. Edit `custom_char_fields.rb` (in `plugins/profile`) with the below. ANOTHER NOTE: if you have already added custom tabs to your profile edit screen, you will want to add these fields to your existing data.
```
def self.get_fields_for_editing(char, viewer)
        return {
          looking_for_rp_announce: char.looking_for_rp_announce == "on" ? true : false ,
        }
      end
```
8. Also in `custom_char_fields.rb`, edit the custom hook to include the below :
```
def self.save_fields_from_profile_edit2(char, enactor, char_data)
        char.update(looking_for_rp_announce: Website.format_input_for_mush(char_data["custom"]["looking_for_rp_announce"] == true ? "on" : "off"))
      end
```
9. Add these lines to your custom styles:
```
.lfrp-row {
  display: flex;
  flex-direction: row;
  flex-wrap: wrap;
  align-items: center;
  justify-content: space-between;
  margin-right: 8px;
  border-bottom: 2px solid rgba(156, 156, 156, .5);
}

.lfrp-row .fa-solid {
  color: #45a366;
  font-size: .9em;
  padding-left: 15px;
  padding-top: 27px;
}

.lfrp-row .fa-stack {
  margin-right: -17px;
}

.fa-mobile-retro::before {
	background-color: black;
	border-radius: 5px;
}

.badge-sidebar-bgs {
  padding-left: .42em;
  padding-right: .22em;
  padding-top: .32em;
  padding-bottom: .22em;
  margin-right: 3px;
  border-radius: .3em;
  transition: background-color .5s;
}

.badge-sidebar-bgs:hover {
  background-color: rgba(255, 238, 225, 0.805);
  padding-left: .42em;
  padding-right: .22em;
  padding-top: .32em;
  padding-bottom: .22em;
  margin-right: 3px;
  border-radius: .3em;
}
```
10. Set the background-color for the `hover` to something that pleases your eye. This is what the button will do when you mouse over it.
11. Set the color for the `.lfrp-row .fa-solid` to something that pleases your eye. This is what the phone icon will be colored (if you have the Text plugin). 
12. Add the contents of the following files from the `custom` folder to your versions of those files in the Components folder. If you have no custom sidebar/play screen data, you can copy-paste the entire file.
	- `sidebar-custom.hbs`
	- `sidebar-custom.js`
	- `play-custom-sidebar.hbs`
	- `play-custom-sidebar.js`
13. Edit `char-edit-custom-tabs.hbs` to add this line.
```
<li><a data-bs-toggle="tab" class="nav-link" href="#lfrp">Looking for RP</a></li>
```
14. Edit `char-edit-custom.hbs` to add this segment:
```
<div id="lfrp" class="tab-pane fade in">

Announce to the game when Looking for RP?

<Input @type="checkbox" @checked={{this.char.custom.looking_for_rp_announce}} />

</div>
```
15. Edit `char-edit-custom.js` so that the existing segment includes the new return line:
```
onUpdate: function() {
    // Return a hash containing your data.  Character data will be in 'char'.  For example:
    // 
    // return { goals: this.get('char.custom.goals') };
    return { looking_for_rp_announce: this.get('char.custom.looking_for_rp_announce')
    };
  }
```
16. After pulling your changes onto the game, do the following, in order:
	- `load config`
	- `load website`
	- `load profile`
	- `load styles`
	- `website/deploy`

Following a successful load and website deploy, the installation should be complete and a hard refresh will show the new sidebar section. Use this opportunity to tweak the CSS to your liking.

## Uninstalling

Removing the plugin requires removing all of the associated database fields and undoing all of the edits made to the web portal files.

## License

Same as [AresMUSH](https://aresmush.com/license).