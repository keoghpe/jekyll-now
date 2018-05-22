---
layout: post
title: Better CSV Specs
---

I've always found testing CSVs with RSpec to be pretty painful. Generally, CSVs are tested like this:

```
expect(my_row).to eq(['First Name', 'Last Name'])
```

This has two major downsides:

- The spec becomes harder to read and maintain as the number of rows and columns you are interested in testing grows.
- When the test fails, the output RSpec gives you can be hard to read or even useless. Often it can look like the following:

```
expected: ["elements", "at", "the", "start" "..., "elements", "at", "the", "end"]
     got: ["elements", "at", "the", "start" "..., "elements", "at", "the", "end"]

(compared using ==)
```

This isn't helpful when the difference is somewhere in the middle of the row!

Recently, I've created a helper which looks like this:

```ruby
module ReportsSpecHelper
  def expect_csv_to_equal(expected, actual)
    expected_titles = expected.first

    expected.each_with_index do |row, row_index|
      row.each_with_index do |cell, column_index|
        actual_value = actual[row_index][column_index]
        expect(actual_value).to eq(cell), lambda {
          value_for_string = if row_index > 0
                               "value for #{expected_titles[column_index]}"
                             else
                               "value"
                             end
          "Cell #{row_index}:#{column_index} (#{actual_value}) did not match expected #{value_for_string} (#{cell})."
        }
      end
    end
  end
end
```

I use this to compare the CSV under test with a CSV I have stored in the same directory as the spec.
It iterates every row and column in the CSV and performs a comparison. 

This has two advantages:

- I can read and edit the expected CSV in a CSV editor (Rubymine has one built in).
- If it fails I get a nice error message:

```
"Cell 2:1 (Peter) did not match expect value for First Name (Pete)"
```


