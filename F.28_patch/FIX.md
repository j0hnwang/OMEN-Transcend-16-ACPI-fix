# F28 Patch
**Disclaimer**
I am not an expert in ACPI or DSDTs, this came from a lot of testing and use of AI. These edits may work for you as they worked for me, but I'm really in no position to help anyone.

## State of Linux on HP OMEN Transcend 16
On my personal computer, using CachyOS, Grub2, and proprietary Nvida drivers, everything seems to be working fine. Speakers, touchpad, and Nvidia GPU seem to be fine (with a small caveat I'll explain later). The only remaining issue I have is with fans. I do not have access to seeing fan rpm or controlling fans in anyway. This could be user error at least for accessing. Im not sure to what degree the BIOS actually lets you mess with fans. That being said, the fans work automatically so lack of access and control is not an issue for me.

**If you have a good solution for this fan issue, make an issue or pull request.**


## Firstly, make the edits from F.27 provided by no-hands.
*I would recommend adding comments like `// Edit #n`, as you will have to jump back to these locations, so its far easier to navigate by searching for edits.*

## Edit #1
At the location of Edit #3 for F.27, you need to make these changes
```
Method (_INI, 0, NotSerialized)  // _INI: Initialize
{
    /* [Edits] */
    // Change #3
    
    // New to F28
    INT1 = GNUM (GPDI) 
    INT2 = INUM (GPDI)
    // End to F28

    // Set the "enabled" bit (I2CN) to true
    Or (\_SB.PC00.I2C0.I2CN, One, \_SB.PC00.I2C0.I2CN)
    // Steer firmware to ELAN07CA path on 0x15 at 400kHz, using GPIO IRQ
    Store (0x05, \TPDT)    // branch that leads to ELAN mapping in your table
    Store (0x15, \TPDB)    // 7-bit I2C address 0x15
    Store (One,  \TPDH)    // ELAN subselector
    Store (One,  \TPDS)    // speed selector: 0=100k, 1=400k, 2=1MHz → use 400k
    Store (Zero, \TPDM)    // 0 => use SBFG (GPIO) in _CRS, not GSI -> this handles interrupts

    // If firmware left placeholder HID, force ELAN vendor HID
    If (LEqual (_HID, "XXXX0000"))
    {
        Store ("ELAN07CA", _HID)
    }
    /* [End Edits] */
```

## Edit #2
At the location of Edit #4 from F.27, `Store (0x02, \TPDS)` needs to be changed to `Store (One, \TPDS)`


## Additional edits
These seem to conclude the necessary changes. 

A few lines above the change to Edit #3 (*was line 107610 for me*), at the line
`GpioInt (Level, ActiveLow, ExclusiveAndWake, PullDefault, 0x0000,`, I had changed `Level` to be `Edge`, but it seems the compiler switched it back. So for completeness try this if the other changes are still not working.


## GPU and Power Saving
On my system, the Nvidia GPU wasn't fully sleeping. So even if I wasn't using it, it was pulling a ton of power (my system would idle around 55-60 W). The work around I found was using `envycontrol` and `powerprofilesctl` to setup aliases for power states. Switching my GPU off and setting to power-saver when I didn't need the Nvidia GPU, I could drop my energy consumption to about 21-24 W when idling / web browsing. This requires a full reboot each time I switch states, which for me takes about a minute. Not a required fix but worth mentioning for anyone trying to make Linux a daily driver.


**Good luck. If you have any comments, fixes, or similar, please make an issue or discussion on the repository. More eyes and voices on this issue will help the community.**

