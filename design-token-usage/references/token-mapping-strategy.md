# Token Mapping Strategy

## 1. Match by semantic role first

Before comparing raw values, ask what job the literal is doing in context —
this catches cases where the *nearest value* would be misleading:

- Is it a `.background(...)`? Prefer a `Surface` role token.
- Is it a `.stroke(...)`/`.border(...)`? Prefer a `Border` role token.
- Is it applied to text/foreground? Prefer a plain neutral or brand token, not
  a state-color role variant.
- Does the surrounding view represent an error/success/warning/info state
  (an alert, a validation message, a status badge)? Prefer the matching
  semantic group (`Danger`/`Success`/`Warning`/`Info`) over a neutral.

## 2. Fall back to nearest value

If the role isn't obvious from context, compare the literal's actual value
against every candidate token's value:

- **Colors**: compute RGB (or perceptual) distance against each token's actual
  color value; propose the closest one only if the distance is small enough to
  plausibly be the same intended color (not just the least-bad option among
  unrelated ones).
- **Fonts**: compare the literal point size against each typography token's
  defined size; propose the closest one only if it matches a token's size
  *and* the token's weight is compatible with the literal's weight — don't
  match on size alone if the weight clearly diverges.

## 3. Mark unmatched when neither works

If role and value both fail to produce a confident single candidate — e.g.
the literal's value doesn't cluster near any existing token — mark the finding
**unmatched** and say why (e.g. "no border token within a plausible distance of
this exact color; may need a new token, which is outside this skill's scope").
Never pick the "least wrong" token just to close out a row in the findings
table — a wrong replacement is worse than an honest "needs a human."

## Worked example (hris-ios)

`HRIS/Common/Components/Alert/BaseAlert.swift`'s six `Color.white` uses are
all backgrounds inside alert/dialog surfaces — role match: `Surface` tokens
under whichever semantic group each alert represents (e.g. `Neutral` for a
plain alert, `Danger.Surface` for a destructive-confirmation alert). Don't
default all six to the same token without checking each alert's actual
semantic purpose first.

`HRIS/Common/Enum/TeamAttendanceType.swift:47`'s
`Color(red: 0.9, green: 0.71, blue: 0)` is a status-indicator color with no
obvious matching token in the existing `Danger/Success/Warning/Info` set at
the time of writing — a case where value-matching found no plausible
candidate. This should be reported unmatched, not forced into the closest
(and clearly wrong) semantic group.
