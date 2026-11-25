# Onear Acoustic Modem

Onear transfers text and files between nearby devices using their speakers and microphones. The browser app runs from a single HTML file and needs no server for local playback. Microphone access requires HTTPS, localhost, or a local file URL supported by the browser.

## Use

Open `index.html` in a current browser, or open the hosted page over HTTPS. On the receiving device, choose **Listen** and allow microphone access before starting a transfer.

Choose **Fast** for a single transmission pass with one parity packet per group of four data packets. Choose **Confirmed delivery** when the receiver should request missing packets and acknowledge a complete file. Both devices must be listening to each other for the confirmation mode.

Text and files up to 32 KiB are supported. The receiver checks the reconstructed content with SHA-256 before showing it as verified or making a file available to download.

The optional frequency-band selector offers wide, low, and high audio profiles. The receiver detects the profile from its synchronization chirp. Device speakers, microphones, distance, orientation, and room acoustics affect the link.

## Transfer pipeline

The sender encodes text as UTF-8 or packages a file name and file bytes with its length and SHA-256 digest. It divides the resulting byte stream into 128-byte payload blocks. Each packet includes a transfer identifier, packet type, sequence number, block count, and CRC-32.

For each group of four data packets, the sender also creates an XOR parity packet. Confirmed delivery adds a second, weighted parity packet. The receiver can reconstruct up to one missing data packet in Fast mode, or up to two missing packets in Confirmed delivery mode, provided the parity packets arrive.

The audio layer uses a chirp for synchronization and OFDM with 64 QPSK carriers. A cyclic prefix helps tolerate echoes. Each audio symbol carries eight bytes protected by a shortened Hamming code. The receiver searches for the synchronization chirp, identifies the selected frequency band, demodulates the carriers, corrects supported bit errors, and rejects packets that fail CRC-32.

In Confirmed delivery mode, the receiver requests missing packet numbers after a pass. The sender retransmits the requested data packets and the end marker. Once the receiver reconstructs the original bytes and verifies SHA-256, it sends an audio acknowledgement. The sender makes up to three attempts and reports when delivery could not be confirmed.

## Frequency bands

All profiles use a 48 kHz audio sample rate.

| Profile | OFDM size | Active carrier range |
| --- | ---: | ---: |
| Wide | 512 | 3.94–9.84 kHz |
| Low | 1024 | 0.94–3.91 kHz |
| High | 1024 | 14.53–17.53 kHz |

These are selectable frequency ranges, not a measured ranking of range, comfort, or reliability across devices.

## Performance

The app currently estimates a transfer at roughly **150 bytes/s** in the wide-band profile. For example, a 14,437-byte file is shown as about 96 seconds per pass. This is a calculated audio-time estimate from packet count, parity overhead, and symbol duration, not a byte-verified measurement between physical devices. Real end-to-end throughput depends on the selected profile, device audio hardware, missed packets, microphone startup, receiver processing, acknowledgements, and retransmissions.

Published figures from other projects provide context, not a direct benchmark:

| Project | Published result | Conditions |
| --- | ---: | --- |
| Onear Acoustic Modem | About 150 bytes/s estimated | Wide-band audio-time estimate shown by the app; not measured over the air |
| [ggwave](https://github.com/ggerganov/ggwave) | 8–16 bytes/s | Library estimate, depending on protocol parameters |
| [Quiet.js](https://github.com/quiet/quiet-js) | About 7 kbit/s audible; about 64 kbit/s over cable | Project profile figures; cable mode is not speaker-to-microphone transmission |
| [Cyrinx](https://github.com/dweekly/cyrinx) | 65.875 kbit/s scheduled goodput, Mac to Pixel | Specific MacBook Pro and Pixel 7a setup; 4,215 of 4,280 blocks verified. The report notes evidence and verifier limitations |

ggwave is an earlier reference for practical data-over-sound software. This project uses FFT-based OFDM and parity coding as established digital-communications techniques. The implementation does not claim that ggwave uses these same techniques.

## Further reading

- [ggwave](https://github.com/ggerganov/ggwave) for a compact data-over-sound library.
- [Quiet.js](https://github.com/quiet/quiet-js) for browser audio transmission profiles, synchronization, and forward error correction.
- [Cyrinx](https://github.com/dweekly/cyrinx) for measured OFDM performance, channel estimation, adaptive profiles, and device-level validation.

## Limits

The current interface supports text and files up to 32 KiB. A transfer cannot be guaranteed when the receiver does not return an acknowledgement. In that case the sender reports delivery as unconfirmed. Acoustic performance depends on the actual device pair and environment.
