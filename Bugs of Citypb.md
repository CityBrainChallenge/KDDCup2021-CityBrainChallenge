# Bugs of Citypb

1.在车刚产生的时候，调用get_vehicle_info,会在

`info["intersection"] = v->vehicle_status_.next_signal_->unique_id_;`
这一行出现问题（段错误）。而且相同代码，有的时候会错误，有的时候没问题。（怀疑是指针问题）