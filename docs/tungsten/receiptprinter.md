# Receipt Printer in Tungsten Self-Checkouts

Receipt printer support in tungsten allows easy printing with error handling through the Built-in Printer API.

## Purposes of receipt printer

- Saving and keeping track of the transaction data
- Printing receipts, reports and staff log-in barcodes
- Handle error reporting: Detect and report printer errors (e.g., paper jams or low paper)

## Example receipt printer code

```lua
local status = {
    paper = "ok" -- ok, warning, fault
}

script.Parent.Parent.InternalCommunication.OnInvoke = function(device, request, data)
    if device == "ReceiptPrinter" then
        if request == "GetStatus" then
            return {["success"] = true, ["error"] = "", ["requires_intervention"] = false, ["response"] = status}
        elseif request == "PrintDocument" then
            --[[
                in this case, data is an array of rows to print
                e.g.:
                {
                    {["l"] = "", ["c"] = "Store", ["r"] = "", isSeparator = false, lineSize = 14},
                    {["l"] = "", ["c"] = "St. Street 1", ["r"] = "", isSeparator = false, lineSize = 14},
                    {["l"] = "", ["c"] = "Receipt", ["r"] = "", isSeparator = false, lineSize = 20},
                    {["l"] = "", ["c"] = "", ["r"] = "", isSeparator = true, lineSize = 14},
                    {["l"] = "Apple 1 x 0.59", ["c"] = "", ["r"] = "0.59", isSeparator = false, lineSize = 14},
                    ...
                }
            ]]

            if status.paper == "fault" then
               return {
                    ["success"] = false, 
                    ["error"] = "Receipt empty",
                    ["requires_intervention"] = true,
                    ["response"] = {
                        ["generalized_instructions_data"] = {
                            ["error_description"] = `Receipt empty. Open printer, replace paper roll and press "continue".`,
                            ["status_value"] = {"paper", "open_cover"},
                            ["expected_value"] = {["paper"] = {"ok", "warning"}, ["open_cover"] = {"ok"}}
                        }
                    }
                }
                -- If requires_intervention is true, the checkout will keep checking the status  
                -- until the issue is resolved (status changes to "ok" or "warning").
            end

            return {["success"] = true, ["error"] = "", ["requires_intervention"] = false, ["response"] = {}}
        end
        return {
            ["success"] = false, 
            ["error"] = "unknown request",
            ["requires_intervention"] = false,
            ["response"] = {}
        }
    end
end

```

!!! warning
    The checkout will remain in intervention mode until the printer reports "ok" or "warning" for each value.  
    Responding with any other status will prevent the system from proceeding.
    Additionally, when requesting printer status, if **any field** in the returned status table  
    has a value of "fault" or an unrecognized value (i.e., not "ok" or "warning"),  
    the system will require an intervention, even if `requiresIntervention` is set to false

!!! INFO  
    If any field in the status table has the value "warning", the trilight on the device will flash green, indicating a device warning state. Additionally, the checkout status in RAP will show "Low media".  

### Explanation

1. **Printer status handling**

    - The `status` table keeps track of various printer states (e.g., `paper`, `toner`, `cover`).

    - Each `status` field can have values such as `ok`, `warning`, or `fault`.

    - The `GetStatus` request returns the current printer status.

    - If any status field is `fault`, the checkout system enters intervention mode until the issue is resolved.

2. **Printing Documents**

    - The `PrintDocument` request processes an array of receipt lines.

    - Each line object contains:

        - `l`: Left-aligned text

        - `c`: Center-aligned text

        - `r`: Right-aligned text

    - `isSeparator`: Boolean indicating whether the line is a separator

    - `lineSize`: Font size of the line

    - If any field in status is `fault`, an error response is returned requiring intervention.

## Error Handling & System Behavior

- The system remains in intervention mode if any printer status field is `fault`.

- Status fields may include `paper`, `toner`, etc.

- If any field in the status table contains `warning`, the device’s trilight will flash green, signaling a media warning state.

- The checkout status in RAP (Remote Attendant Program) will display corresponding messages (e.g., "Low media").

- A `requires_intervention = true` response means the system will keep checking the status until resolved.

- The intervention prompt provides guidance for refilling the paper, replacing toner, or resolving other errors.

## Sending intervention with instructions

To notify the system that a printer issue requires manual intervention, return a structured response with detailed instructions. Below is an example of handling a paper jam intervention:

### Detailed instructions

``` lua
return {
        ["success"] = false, 
        ["error"] = "Empty paper",
        ["requires_intervention"] = true,
        ["response"] = {
            ["detailed_instructions_data"] = {
                ["images"] = {
                    [1] = "rbxassetid://",
                    [2] = "rbxassetid://",
                    [3] = "rbxassetid://",
                    [4] = "rbxassetid://",
                },
                ["error_description"] = "Receipt empty",
                ["error_code"] = "PTRSTAT_REC_EMPTY - 0x00000002",
                ["error_solution"] = [[
                    1. Unpack the receipt paper.
                    2. Unlock and open the upper cabinet door.
                    3. Click the receipt printer to remove any 
                    excess thermal paper and load a new roll.
                    4. The printer automatically presents and
                    cuts a receipt. A test smiley face should display 
                    on the receipt. A “smiley face” prints on the
                    test receipt to indicate that the thermal coating 
                    is on the side where the thermal print head 
                    is located. When the smiley face prints, the
                    paper is loaded correctly.
                    5. Close and lock the upper cabinet door.
                ]],
                ["status_value"] = {"paper", "open_cover"},
                ["expected_value"] = {["paper"] = {"ok", "warning"}, ["open_cover"] = {"ok"}}
            }
        }
    }
```

#### Detailed response breakdown

- `success = false`: Indicates the request failed due to an error.

- `error`: Describes the issue (e.g., "Paper jam").

- `requires_intervention = true`: Forces the system into intervention mode.

- `detailed_instructions_data`:

    `images`: List of asset IDs for instructional images (must be exactly 4 or none).

    `error_description`: A short explanation of the problem.

    `error_code`: A unique identifier for the error.

    `error_solution`: Step-by-step instructions to resolve the issue.

    `status_value`: The current faulty status in the status table.

    `expected_value`: The status value that will indicate the issue is resolved.

### Generalized instructions

``` lua
return {
        ["success"] = false, 
        ["error"] = "Paper jam",
        ["requires_intervention"] = true,
        ["response"] = {
            ["generalized_instructions_data"] = {
                ["error_description"] = `Paper jam. Open printer, adjust paper and press "continue".`,
                ["status_value"] = {"paper_jam", "open_cover"},
                ["expected_value"] = {["paper_jam"] = {"ok", "warning"}, ["open_cover"] = {"ok"}}
            }
        }
    }
```

#### Generalized response breakdown

- `success = false`: This indicates that the request has failed due to an error.

- `error = "Paper jam"`: This string describes the specific issue the system is encountering, in this case, a "Paper jam".

- `requires_intervention = true`: This flag signals that the system needs manual intervention to resolve the issue.

- `generalized_instructions_data`: This is a container for the essential steps and data related to the error.

    `error_description`: A concise explanation of the problem and the action needed to fix it.

    `status_value`: This array lists the current faulty statuses that indicate the error and needs to be fixed.

    `expected_value`: This defines the statuses that indicate successful resolution of the issue. After resolving the paper jam and closing the cover, the status values should reflect:

    - `ok` or `warning` for the `paper_jam` status (indicating that the issue is cleared or acknowledged),

    - `ok` for the `open_cover` status (indicating that the cover is closed).
